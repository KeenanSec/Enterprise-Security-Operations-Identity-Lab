## Reset Linkwarden password

Forgot my Linkwarden password and there was no working web reset, so I reset it directly in Postgres.

Ref: https://github.com/linkwarden/linkwarden/issues/806

1. Go to the Linkwarden directory.
```bash
   cd /opt/linkwarden
```

2. In the compose file, comment out the `image:` line and uncomment the `build:` line.
   ![Pasted image 20260623225015](../Nginx%20Proxy%20Manager/Images/Pasted%20image%2020260623225015.png)

3. Find the Postgres password.
```bash
   cat .env
```

4. Expose the DB port so I can reach it from the host. Add to the `postgres` service in the compose file, then recreate the stack.
```yaml
   services:
     postgres:
       ports:
         - 37194:5432
```
```bash
   docker compose up -d
```

5. Connect as the postgres superuser to confirm access.
```bash
   PGPASSWORD=[PostgresPassword] psql -h localhost -p 37194 -U postgres -d postgres
```

6. Get the Linkwarden DB connection string from .env.
```bash
   grep -i database /opt/linkwarden/.env
```

7. Connect as the linkwarden user and reset the password.
```bash
   psql "postgresql://linkwarden:[dbpassword]@localhost:37194/linkwardendb"
```
```sql
   CREATE EXTENSION IF NOT EXISTS pgcrypto;
   UPDATE "User" SET password = crypt('TempPass123', gen_salt('bf', 10)) WHERE username = '123';
   SELECT username, left(password, 7) FROM "User" WHERE username = '123';
```

8. Remove the exposed port mapping from the compose file and recreate the stack so the DB isn't reachable from the host anymore.