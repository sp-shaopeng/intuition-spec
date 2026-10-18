# Local Stack — Docker Compose Runbook

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

> **2026-09-05 correction:** Local configuration examples do not define the Amazon Web Services production topology. Retired encryption keys are not a requirement for user email/phone; retain only settings actually needed by the chosen local configuration. [Current requirements](../CURRENT-REQUIREMENTS.md). No runtime verification was performed by this documentation correction.

How to bring up the entire InTuition SG stack on your machine with one command.

## What you get

A4-service local development Compose stack. Amazon Web Services production topology is defined separately by027:

| Service  | Image                          | Host port           | Purpose                                                |
|----------|--------------------------------|---------------------|--------------------------------------------------------|
| postgres | postgres-alpine           | REDACTED ip address     | Application database. Liquibase migrates on first boot |
| redis    | redis-alpine (AOF, authed) | REDACTED ip address     | Rate limit buckets, email stream, multi-factor authentication session cache    |
| backend  | Spring Boot 3.3 / Java 21      | REDACTED ip address     | REST application programming interface                                               |
| frontend | nginx serving the Vite SPA     | REDACTED ip address     | React user interface                                               |

All host ports are bound to REDACTED ip address so the stack does not leak the dev secrets onto your LAN.

## Prerequisites

- **Docker Desktop** running (it ships the Compose plugin)
- 6 GB of free disk space (Maven layer cache + node modules + Postgres data)
- Ports 5432, 6379, 8080, 5173 free on REDACTED ip address

That's it. You don't need Java, Maven, Node, or any other tool locally — every build step happens inside Docker.

## First-time setup


Implementation example omitted. This public edition retains the written requirements in this section.


The first invocation will:

1. Pull postgres-alpine, redis-alpine, maven.9-eclipse-temurin-21, eclipse-temurin-jre-alpine, node-alpine, nginx.27-alpine (~600 MB cold).
2. Build the backend image — multi-stage:
   - Stage 1: the documented operational check (warms layer cache), the documented operational check
   - Stage 2: copy the jar onto an Alpine JRE
3. Build the frontend image — multi-stage:
   - Stage 1: the documented operational check with VITE application programming interface BASE URL is http://localhost baked in
   - Stage 2: copy dist/ onto nginx
4. Start the stack respecting healthcheck dependencies:
   - postgres and redis start in parallel
   - backend waits until both are healthy (pg isready + the documented operational check)
   - the frontend waits until the backend health check succeeds

Cold first boot end-to-end is ~3-5 minutes on a recent Mac Book. Subsequent boots reuse cached layers and are ~30 seconds.

### Detached mode

Append d to run in the background. Logs are still available via the documented operational check.


Implementation example omitted. This public edition retains the written requirements in this section.


## Liquibase on first boot

By default application implementation reference has spring liquibase.enabled is no because production runs migrations as a Railway release-command pre-deploy step. The Compose file **overrides** this via the SPRING LIQUIBASE ENABLED is yes env var on the backend service, so on first boot against a clean postgres data volume the backend will:

1. Acquire the databasechangeloglock row in Postgres.
2. Apply all 17 changesets (0001-create-users implementation reference through 0017-add-cancelled-invoice-status implementation reference) in order.
3. Release the lock.
4. Continue Spring Boot startup.

You will see liquibase executor.jvm Jdbc Executor log lines during this phase. The whole migration takes ~10-20 seconds on first boot.

Verify migrations applied:


Implementation example omitted. This public edition retains the written requirements in this section.


Expect 17 rows.

## Healthcheck verification


Implementation example omitted. This public edition retains the written requirements in this section.


All four services should show healthy within ~120 seconds of cold boot. The backend has start period: 120s to cover JVM warmup + Liquibase.

If backend stays unhealthy:


Implementation example omitted. This public edition retains the written requirements in this section.


Most common cause: a Liquibase changeset failed (typo in XML, FK violating an existing row). Look for liquibase exception.Liquibase Exception.

## Smoke test


Implementation example omitted. This public edition retains the written requirements in this section.


## Useful commands


Implementation example omitted. This public edition retains the written requirements in this section.


## Hot reload

Compose builds bake the code into images. For an inner-loop dev experience:

- **Backend**: edit Java locally, then the documented operational check, then the documented operational check. Or, run the backend natively against the containerized Postgres + Redis:


Implementation example omitted. This public edition retains the written requirements in this section.


- **Frontend**: the documented operational check locally (proxy to http://localhost). Stop the frontend container or just ignore port 5173 on the container side.

## Tearing down


Implementation example omitted. This public edition retains the written requirements in this section.


## Configuration via .env

Copy the example and edit:


Implementation example omitted. This public edition retains the written requirements in this section.


Compose reads .env implementation reference automatically. Variables you might want to override:

| Key                          | Default                                  | Reason to change                |
|------------------------------|------------------------------------------|---------------------------------|
| VITE application programming interface BASE URL          | http://localhost                  | Point frontend build at a remote application programming interface  |
| cross-origin request permissions ALLOWED ORIGINS       | http://localhost                  | Add a tunneled origin           |

The dev defaults for signed session token SECRET CURRENT, keyed message authentication SECRET KEY, and MASTER ENCRYPTION KEYSET are hard-coded in docker-compose implementation reference because rotating them would also invalidate any data already encrypted in your Postgres volume. **Never reuse these values in any non-local environment.**

The root .gitignore blocks .env implementation reference from being committed; only .env implementation reference is tracked.

## Troubleshooting

**Bind: address already in use on 5432, 6379, 8080, or 5173**
You have a local Postgres/Redis/etc. running. Either stop it (brew services stop postgresql) or change the host port in docker-compose implementation reference.

**Backend stays unhealthy for >3 minutes**
Tail the logs (the documented operational check). Typical causes:
- Liquibase migration failed — fix the changeset and the documented operational check.
- Tink keyset rejected — confirm the MASTER ENCRYPTION KEYSET env in docker-compose implementation reference is a valid AES-SIV keyset JSON.
- Redis auth failed — confirm REDIS PASSWORD value (withheld) matches the password baked into the redis service command.

**Cannot connect to the Docker daemon**
Docker Desktop is not running. Open Docker Desktop and wait for the whale icon to stop animating, then retry.

**Frontend shows a cross-origin request permissions error when calling the application programming interface**
The compose file sets cross-origin request permissions ALLOWED ORIGINS is http://localhost on the backend. If you renamed the frontend host port or tunneled it through another origin, add that origin to cross-origin request permissions ALLOWED ORIGINS.

**XSRF-TOKEN cookie missing on state-changing requests**
The SPA calls read operation for auth csrf at app boot via use Init Auth implementation reference. If you bypassed that (e g., calling the application programming interface directly with the documented operational check), bootstrap first:


Implementation example omitted. This public edition retains the written requirements in this section.


**Wanting to run integration tests against the stack**
The backend has Testcontainers tests that spin up their own ephemeral Postgres+Redis — they do not need the Compose stack and will fight it if both try to claim ports. Stop the Compose stack before running the documented operational check natively.

## File map

| Path                                       | What it does                                                           |
|--------------------------------------------|------------------------------------------------------------------------|
| docker-compose implementation reference                       | The stack definition. Edit cautiously.                                 |
| .env implementation reference                             | Copy → .env implementation reference to override defaults                                     |
| .gitignore                               | Blocks .env implementation reference and other untracked artifacts                            |
| intuition-the private implementation folder | Multi-stage build for the dev backend image                            |
| intuition-the private implementation folder     | Production image (assumes jar is pre-built in target/)               |
| intuition-the private implementation folder            | Multi-stage build for the frontend static-serve image                        |
| intuition-the private implementation folder            | SPA routing + immutable asset cache + security headers                 |
