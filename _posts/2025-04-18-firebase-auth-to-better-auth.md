---
layout: post
date: 2025-04-18 12:50:00
title: "How I Migrated from Firebase Auth to Better Auth Without Downtime or Bulk User Imports"
tags: [Better-Auth, Firebase Auth, Postgres, Next.JS]
# image: "/images/2020/10/cookies.jpg"
published: true
comments: true
---

# How I Migrated from Firebase Auth to Better Auth Without Downtime or Bulk User Imports

I recently migrated my SaaS app from Firebase Auth (Google login + email/password) to [Better Auth](https://www.better-auth.com/). I didn’t export thousands of users, didn’t force password resets, and had **zero downtime**. Instead, I let users migrate naturally by logging in.

Here’s how I did it, why I did it, and what I learned along the way.

---

## Why I Left Firebase Auth

Firebase Auth worked at the beginning. But as the app matured, it became a burden. Here’s why:

- **Too complex for simple auth:** I only needed Google login and email/password. Firebase’s SDKs, ID tokens, and admin setup started to feel like overkill.

- **Not a good fit with Next.js App Router:** Getting the current user in Server Actions or Route Handlers required verifying Firebase ID tokens manually using the admin SDK. Managing auth state across client and server was clunky and error-prone.

- **Two sources of truth:** I had user data in Firebase and also in Postgres. Every sign-up required syncing between the two. That’s not ideal.

- **Limited flexibility:** Adding multi-tenancy, managing roles, customizing login flows — all that required layers on top of Firebase.

In short, Firebase was a powerful system I no longer needed.

---

## What I Switched To

I chose **Better Auth**, a new TypeScript-native authentication library. It gave me:

- **Database-first auth:** All credentials and sessions live in my own Postgres database (RDS). I use Drizzle as my ORM.

- **Out-of-the-box multi-tenancy:** Better Auth has a plugin that adds `organization` and `member` tables, exactly what I needed.

- **Good Next.js integration:** It works well with App Router and Server Actions. No more juggling between client SDKs and server APIs.

- **Modern features (if I want them):** Like passkeys, passwordless login, 2FA — though I didn’t need these right away.

I’m using:

- **Next.js 15 (App Router)**
- **Drizzle ORM**
- **PostgreSQL (RDS)**
- **Vercel**

---

## Schema Challenges

My app already had:

- A `user` table, using Firebase UIDs as primary keys
- An `organization` table
- A `user_orgs` join table for org memberships

But Better Auth expected:

- A `user` table with specific fields (e.g. `emailVerified`, `image`)
- An `account` table to track logins (Google, email/password, etc.)
- A `member` table for org membership (part of its organization plugin)

### What I did:

- To keep the `user` and `organization` tables simple (with just what Better Auth expects), I:
  1. I adjusted the schema to match what Better Auth expected
  2. Added missing fields like `emailVerified` and `image` to `user`
  3. I created the tables `user_extra` and `organization_extra` and moved the fields I had before (both with one-to-one relationships with their basic counterparts)
  4. Replaced `user_orgs` with `member` table (added an `id` primary key)
  5. Changed the app to use the new tables
- `npx @better-auth/cli@latest generate` to generate the schema in a separate file `auth-schema.ts`
- The `account` table was created by Better Auth

In my `schema.ts` file, I did this:

```ts
import {
  account,
  invitation,
  member,
  organization,
  session,
  user,
  verification,
} from "./auth-schema";

export const accountT = account;
export const invitationT = invitation;
export const memberT = member;
export const sessionT = session;
export const verificationT = verification;
export { verification };
```

So that other tables could set relationships with the new auth tables created by Better Auth, and to be exported as <tableName>T to avoid name conflicts.

One limitation I hit: Better Auth doesn’t support Postgres schemas (namespaces). I had to keep all auth tables in the default schema. Not a big deal, but worth knowing.

---

## How I Migrated Without Importing Users

I didn’t download or import all Firebase users. That sounded messy and risky.

Instead, I let users migrate on demand. The idea was simple:

1. Keep Firebase Auth active temporarily
2. Let Better Auth try to log users in
3. If that fails, fall back to Firebase
4. If Firebase login works, create the user in Better Auth

### Google Login

For Google login, I used Better Auth’s signIn.social("google"). Here’s what happened when a user logged in:

- If they were new: Better Auth created a user + account
- If they already existed in my DB: Better Auth linked the Google account to their existing user

Better-Auth can auto-link accounts by matching verified emails. Google provides a verified email by default, so this was safe. I didn’t need to write custom logic for this — it just worked.

### Email and Password Login

This one was more involved. Due to the complexity of the fallback logic, I could not use the default client side login.

```ts
const res = await authClient.signIn.email({ email, password });
```

Instead, I had to do that on the backend. Fortunately we are using Next.js App Router, so I could use Server Actions.

Here’s what happened:

1. Check for existing user in your database:

   - Query the user and account tables by email.

   - Use a left join to see if the user has an existing Better Auth account.

2. If user exists in your DB but has no Better Auth account:

   - Try to authenticate with Firebase (legacy system) using the provided email and password.

3. If Firebase login is successful:

   - Get the Better Auth context to access the password hasher.

   - Hash the provided password using Better Auth’s hashing method.

   - Insert a new account record into Better Auth’s account table:

     - Use "credential" as providerId

     - Link it to the existing user.id

     - Set a new accountId, password, and timestamps

4. After migrating (or if already migrated), try to sign in via Better Auth:

   - Call auth.api.signInEmail(...) with email and password.

5. If the Better Auth sign-in fails:

   - Check if the error is an instance of APIError.

   - Try to extract a known error message (e.g., invalid credentials).

   - If it’s a known error, return it in a structured format.

   - If it’s an unknown/unhandled error, rethrow it.

6. If sign-in succeeds:

   - Optionally redirect the user to a specified route (redirectTo).

   - Return the sign-in result.

On the next login, Better Auth handles everything natively.

```ts
export const signInWithEmail = async (
  email: string,
  password: string,
  redirectTo?: string
) => {
  const users = await db2
    .select()
    .from(userT)
    .leftJoin(accountT, eq(userT.id, accountT.userId))
    .where(eq(userT.email, email));

  const user = users[0];

  if (user?.user && !user?.account) {
    // User exists but no account

    const firebaseUser = await firebaseSignIn(email, password);

    if (firebaseUser) {
      // Create account

      const ctx = await auth.$context;
      const hashedPassword = await ctx.password.hash(password);

      await db2.insert(accountT).values({
        id: createId(),
        accountId: createId(),
        providerId: "credential",
        userId: user.user.id,
        createdAt: new Date(),
        updatedAt: new Date(),
        password: hashedPassword,
      });
    }
  }

  const result = await tryCatch(
    auth.api.signInEmail({
      headers: await headers(),
      body: {
        email,
        password,
      },
    })
  );

  if (!result.isSuccess) {
    if (result.error instanceof APIError) {
      const errorMessage = await getErrorMessage(result.error);
      /// Known error
      if (errorMessage) return { ...result, error: { message: errorMessage } };
    }

    // Rethrow the unhandled errors
    throw result.error;
  }
  // Usually "/"
  if (redirectTo) {
    redirect(redirectTo);
  }

  return result;
};
```

## No need to reset passwords, no need to export Firebase password hashes.

## Benefits of This Strategy

- No downtime
- No forced password resets
- No mass import of Firebase users
- Seamless experience for active users
- Firebase fully removed once migration is done and critical mass of users migrated

## After some time, say 180 days, I can safely turn off Firebase. If a user hasn’t logged in by then, I can prompt them to reset their password using the new system.

## Gotchas and Lessons

- Email collisions: Make sure Google logins use the same email as Firebase. Otherwise, you might end up with duplicates.
- Don’t forget emailVerified: Better Auth uses this to decide if it should trust an OAuth login. I defaulted it to true for Google users.
- Test fallback login flow: I caught a few bugs in staging — like retrying the login after inserting the new password.
- Avoid mixing auth states: I briefly tried to decode both Firebase tokens and Better Auth sessions at once — it was messy. I focused only on one system at a time in runtime.

---

## Final Thoughts

This was one of the smoothest migrations I’ve done. I kept things simple:

- I didn’t overthink user imports
- I respected what Firebase was still doing, until it wasn’t needed
- I used the login flow as the migration trigger

The result: a much cleaner, self-contained auth system that lives in my app, in my database, with no external dependencies.

If you're building a SaaS on Next.js and Postgres, Better Auth is worth checking out.

Resources:

- [Better Auth Docs](https://www.better-auth.com/docs)
- [Drizzle ORM](https://drizzle-orm.com/)
- [Firebase Auth REST API](https://firebase.google.com/docs/reference/rest/auth)
- [Optigrid](https://www.optigrid.io/) - Where I made the migration
