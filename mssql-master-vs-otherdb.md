Classic SQL Server gotcha. You can always connect to **master** (public has CONNECT), but your login may not have access to the other DB, or the DB itself isn’t accessible.

Here’s a quick checklist to find/fix it:

1. Verify the database is usable
   Run from master:

```sql
SELECT name, state_desc, user_access_desc, is_read_only
FROM sys.databases
WHERE name = N'<YourDb>';
```

* `OFFLINE/RESTORING/RECOVERY_PENDING/SUSPECT` → fix state first.
* `SINGLE_USER/RESTRICTED_USER` → may block you.
* If it’s a secondary replica or read-only, connect with proper options and permissions.

2. Check the exact error code
   If sqlcmd shows:

* **4060** “Cannot open database requested by the login” → database exists but your login has no user in it (or is denied).
* **18456** with states:

  * State 38/40 → login ok, but default/target DB not accessible.
  * State 2/5 → bad login/ password.

3. Map your login to a user in the target DB (most common fix)
   From master (or after `sqlcmd -d <YourDb>` if you can get in as another admin):

```sql
USE [<YourDb>];
-- If a matching user does NOT exist:
CREATE USER [<Domain\Login or SqlLogin>] FOR LOGIN [<Domain\Login or SqlLogin>];

-- (optional) give minimal read access to test
EXEC sp_addrolemember N'db_datareader', N'<Domain\Login or SqlLogin>';

-- If a user exists but is orphaned (no login mapping):
ALTER USER [<UserNameInDb>] WITH LOGIN = [<Domain\Login or SqlLogin>];
```

4. Ensure you’re actually pointing sqlcmd at that DB

```bash
sqlcmd -S <server>\<instance>,<port?> -E -d <YourDb>
# or SQL login:
sqlcmd -S <server> -U <login> -P "<password>" -d <YourDb>
```

If `-d <YourDb>` fails but `-d master` works → permission/mapping or DB state issue (steps 1–3).

5. If your login’s **default database** is broken/offline
   You might only get in by forcing master:

```bash
sqlcmd -S <server> -E -d master
```

Then fix the default DB:

```sql
ALTER LOGIN [<Domain\Login or SqlLogin>] WITH DEFAULT_DATABASE = [master];
```

6. Contained database users (if enabled)
   If the DB uses contained authentication, you must connect with the **contained user** and `-d <YourDb>`; a server-level login won’t work there:

```sql
-- On the server (once, by an admin):
EXEC sp_configure 'contained database authentication', 1;
RECONFIGURE;

-- In the DB:
CREATE USER [myContainedUser] WITH PASSWORD = 'Strong#Pass123';
EXEC sp_addrolemember N'db_datareader', N'myContainedUser';
```

Then connect:

```bash
sqlcmd -S <server> -U myContainedUser -P "Strong#Pass123" -d <YourDb>
```

7. Check for explicit DENY
   Inside the DB:

```sql
SELECT d.name AS principal, p.permission_name, p.state_desc
FROM sys.database_permissions p
JOIN sys.database_principals d ON p.grantee_principal_id = d.principal_id
WHERE d.name IN (N'<YourLoginOrUser>') AND p.state_desc = 'DENY';
```

Remove DENY if present and intended:

```sql
REVOKE CONNECT TO [<User>];
```

8. Still stuck? Inspect the error log for the real reason
   From master:

```sql
EXEC xp_readerrorlog 0, 1, N'Login failed';
```

The “State = xx” helps pinpoint the cause.

If you paste your exact sqlcmd command and the full error message (including state), I’ll pinpoint which of the above applies and give the exact one-liner fix.
