---
layout: post
date: 2026-06-03 10:00:00
title: "One App, Lot's of Email issues — So I Built Coldletter"
tags: [Email, MJML, SaaS, Side Project]
published: true
comments: true
---

For a while, [Optogrid]({% post_url 2024-02-07-project-optogrid %}) was quietly paying for and babysitting two separate email systems. One provider for transactional email (welcome messages, password resets, receipts) and a different platform for marketing (onboarding sequences, newsletters). Two dashboards, two bills, two mental models — for one small app. It would be fine if it was not painful...

This is the story of why I folded them into one tool, what I built, and where I think the same idea helps other people.

<!-- more -->

## The pains I kept hitting

These weren't hypothetical "wouldn't it be nice" annoyances. They were the friction I felt every time I touched email.

**Two systems for one app.** Transactional lived behind one provider's API. Marketing lived in a separate drag-and-drop platform. They didn't know about each other. A user who unsubscribed in one place was still "active" in the other. Reporting was split in half. And I was paying twice to send email that a single provider's API could already handle.

**Hand-coding email that renders everywhere.** MJML helps, but I was still hand-writing templates and crossing my fingers that they'd survive Gmail, Outlook, and dark mode. No versioning, no review, no diff. When I broke a template, I usually found out the same time a customer did.

**Automation that fought me.** Expressing something as simple as "on signup, send the welcome email, wait a day, then tag the user" took far more clicking than it should in a sprawling visual canvas. And when I hit a wall, I hit it hard — rigid plan limits and features I couldn't bend to my use case. I even filed a bug with one of the tools at some point. Last I checked, it's still open.

**No real control.** The honest version: I wanted email to feel like the rest of my codebase — typed, versioned, testable, in git. Instead it lived in someone else's UI, behind someone else's limits.

## What I wanted instead

- **One API call** to send anything, transactional or marketing.
- **Templates I version and test like code**, with typed variables so a missing field fails loudly *before* it ships, not after.
- **Automations that read like logic**, not a maze.
- And the important one: **keep my own provider.** I didn't want to replace SendGrid or SES — I wanted a layer *on top* of it. Bring your own key. Coldletter rides on your delivery; it doesn't resell it back to you.

## What I built

[Coldletter](https://coldletter.com?utm_source=saulo_blog&utm_medium=link&utm_campaign=coldletter_post) started as Optogrid's internal email service and grew into a standalone product. The shape of it:

- **Versioned MJML/HTML templates** with typed variables and one-click test sends.
- **Campaigns** — one-time bulk sends to a filtered audience.
- **Automations** modeled as trigger → condition → action trees (on subscriber created, send X, wait, branch, tag) instead of a canvas you fight.
- **Compliance built in** — one-click unsubscribe, bounce handling, provider webhooks. The unglamorous stuff you only notice when it's missing.
- **An operator UI** for the people who'd rather not touch the API.

On the light-stack note: a Fastify API with a Postgres-backed job queue (pg-boss) on Fly.io, a Next.js operator console on Vercel, Drizzle on the database, and MJML + Handlebars for rendering. Auth is better-auth — the same library I [migrated to from Firebase]({% post_url 2025-04-18-firebase-auth-to-better-auth %}) on Optogrid.

## What I learned

- **The boring parts are the hard parts.** Cross-client rendering, plain-text fallbacks, one-click unsubscribe, bounce escalation — none of it is glamorous, all of it matters, and all of it is what the "just send email" tools quietly handle for you.
- **Postgres can be the whole backend.** Running the job queue on Postgres instead of reaching for Redis kept the infrastructure small. For a solo build, fewer moving parts is worth a lot.
- **Building internal-first kept me honest.** I generalized to multi-tenant and bring-your-own-key only after the thing was already earning its keep inside Optogrid. Solving my own problem first meant I never had to guess what actually mattered.

## Opportunities for others

I built this for me, but the itch isn't mine alone. It probably sounds familiar if:

- You're paying for a transactional provider *and* a separate marketing tool, and they don't talk to each other.
- You've outgrown an all-in-one platform's limits but don't want to rebuild delivery from scratch.
- You want email templates and automations to live in version control, like everything else you ship.

If that's you, we have the same problem, and [Coldletter](https://coldletter.com?utm_source=saulo_blog&utm_medium=link&utm_campaign=coldletter_post) is my answer to it — take a look. And if you're solving it a different way, I'd genuinely like to hear how.
