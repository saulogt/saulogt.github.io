---
layout: post
date:   2021-08-11 12:50:00
title: "Creating readonly access user in postgress in a single script"
tags: [ Postgres, psql, database, rds]
# image: "/images/2020/10/cookies.jpg"
published: true
comments: true
---

I have an ETL process connected to my RDS Aurora cluster running Postgres.
It automatically detect new tables and sync accordingly to the datawarehouse.
The problem is that recently I noticed it's connecting using the master credential provided by RDS. Obviously it's not very secure approach so I needed to replace the credential with a readonly access.

To avoid manual management of users and permissions I created this script that does:
1. Creates a role named `"readonly"`
1. Grants access to my database
1. Grants access to all schemas
1. Allows `SELECT` on all tables and sequences
1. Sets the default privileges to allow `SELECT` operation. This is particularly important because it must access all new tables

## Usage

1. Save the content of the script bellow to a file
1. Change the values `my_database`, `user1`, and `my_secret_password` to something that makes sense for you
1. Save and run the script

*Important* You cant execute it as a sql code because it variable declaration on the top is invalid SQL. You must use `psql` or another client that supports script execution

```sh
postgres=$ \i /Path/to/your/file.sql
```

## The script

```sql
\set database my_database
\set username user1
\set password '\'my_secret_password\''

CREATE ROLE readonly;

GRANT CONNECT ON DATABASE :database TO readonly;

DO $do$
DECLARE
    sch text;
BEGIN
    
    -- Lists all schemas filtering out system ones
    FOR sch IN SELECT nspname FROM pg_namespace where nspname != 'pg_toast' 
    and nspname != 'pg_temp_1' and nspname != 'pg_toast_temp_1'
    and nspname != 'pg_statistic' and nspname != 'pg_catalog'
    and nspname != 'information_schema'
    LOOP
        EXECUTE format($$ GRANT USAGE ON SCHEMA %I TO readonly $$, sch);
        EXECUTE format($$ GRANT SELECT ON ALL TABLES IN SCHEMA %I TO readonly $$, sch);
        EXECUTE format($$ GRANT SELECT ON ALL SEQUENCES IN SCHEMA %I TO readonly $$, sch);
        
        EXECUTE format($$ ALTER DEFAULT PRIVILEGES IN SCHEMA %I GRANT SELECT ON TABLES TO readonly $$, sch);
        EXECUTE format($$ ALTER DEFAULT PRIVILEGES IN SCHEMA %I GRANT SELECT ON SEQUENCES TO readonly $$, sch);
    END LOOP;
END;
$do$;

CREATE USER :username WITH PASSWORD :password;

GRANT readonly TO :username;
```

### Reference

https://tableplus.com/blog/2018/04/postgresql-how-to-create-read-only-user.html
https://stackoverflow.com/questions/760210/how-do-you-create-a-read-only-user-in-postgresql