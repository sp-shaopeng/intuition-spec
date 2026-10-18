# Deployment Runbook — InTuition → Dev virtual machine (Oracle Cloud)

> Reading note: [Current requirements](../CURRENT-REQUIREMENTS.md) take priority. Named reviewers are AI roles, not employees. Reported test results are historical; see [publication notes](../../docs/publication-notes.md#history-and-review-roles).

Authoritative, step-by-step guide to deploy the InTuition stack to the founder's
remote dev virtual machine in a **same-origin, single-public-port-80** topology (prod-like, no
cross-origin request permissions). For local boot see [runbook md](runbook.md); for the Compose stack reference
see [local-stack md](local-stack.md).

This runbook reflects the deploy performed and verified on **2026-06-04** to
REDACTED ip address. It is repeatable: re-running it re-syncs code and rebuilds.

---

## 0. What this deploys & why this topology

The stack is the same 4-service Compose project as local (postgres, redis, backend,
frontend), but reconfigured so the **browser only ever talks to port 80**:


The illustration for . what this deploys & why this topology contains the following information:

- browser ▶ http://`<virtual machine-IP>`/ (frontend: nginx serves the SPA).
- ▶ http://`<virtual machine-IP>`... interface operation (frontend nginx reverse-proxies → backend).


- **Same-origin** → no cross-origin request permissions, no cross-site cookie problems. The frontend bundle is built
  with an **empty** VITE application programming interface BASE URL, so axios issues **relative** api ... interface operation
  calls that nginx proxies to the backend container on the shared Compose network.
- **Backend stays internal** — not published to the host; reachable only via the frontend
  nginx api interface operation proxy.
- The auth cookie is **host-only** (the backend never sets a Domain attribute — see
  the set auth cookie operation), so it works on a bare IP with no DNS.

---

## 1. Prerequisites

| Requirement | Detail | Check |
|---|---|---|
| virtual machine | Ubuntu 24.04, ≥ 2 vCPU / 4 GB RAM / ~10 GB free | df -h ~, nproc |
| Docker + Compose on virtual machine | Docker ≥ 24, Compose v2+ | the documented operational check, the documented operational check |
| secure remote access key (local) | private key, chmod 600 | stat -f %Sp `<key>` → rw |
| Outbound ports open in OCI | **TCP 22** (secure remote access) must already be allowed | secure remote access works |
| Local stack files | wrapper dir with docker-compose implementation reference + .env implementation reference (has RESEND application programming interface KEY) | ls docker-compose implementation reference implementation reference |

**Connection facts (this deploy):**
- Host: ubuntu@ REDACTED ip address
- Key: `<workspace>` REDACTED key path
- The deploy runs from the **local Mac** and secure remote access/rsyncs to the virtual machine. **Never copy the
  secure remote access key onto the virtual machine.**

> All ssh/rsync/external-the documented operational check commands below run from the local Mac and require
> outbound network (in this harness: dangerously Disable Sandbox: yes).

---

## 2. Transfer the code (rsync, not perform the corresponding private revision-management step

The repos are private, so push the working tree directly. Trailing slashes matter.
The excludes keep secrets/artifacts off the virtual machine (dev Key/, build output, logs).


This example illustrates the . transfer the code rsync, not perform the corresponding private revision-management step


This copies docker-compose implementation reference, .env implementation reference, intuition-backend/, intuition-frontend/,
etc. — **excluding the secure remote access key dir**.

**Verify:** .env implementation reference carries the secrets and dev Key/ did NOT come across:


This example illustrates the . transfer the code rsync, not perform the corresponding private revision-management step


---

## 3. Edit the virtual machine copy ONLY (never the local repos)

Both edits are made in the private deployment directory on the virtual machine. Do **not** touch the local repo files
(no commit, no push — this is a dev-test deploy).

### 3a. Add the api interface operation reverse-proxy to the frontend nginx

Edit the deployed frontend configuration **inside the
the web server configuration block, BEFORE the default request-routing rule** (keep a .bak):


Implementation example omitted. This public edition retains the written requirements in this section.


backend resolves because the frontend container shares the default Compose network.
forward requests to the backend while preserving their full paths with **no** trailing path preserves the full
api ... interface operation path the backend expects.

### 3b. Create docker-compose override implementation reference


Implementation example omitted. This public edition retains the written requirements in this section.


Notes:
- !reset clears the base file's backend port publish (Compose v5.1 supports it —
  validated here). If a Compose version rejects !reset, instead set the backend to
  REDACTED ip address :8080 so it's loopback-only, not public.
- The empty VITE application programming interface BASE URL build arg **must win** over the base file's
  the configured backend address or its local development fallback and over any value in .env implementation reference
  (this deploy's .env implementation reference had VITE application programming interface BASE URL is http://localhost). The explicit
  override arg does win — **verify in step 5** by grepping the built JS.

**Validate the merge before building:**


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 4. Bring up the stack (migrate-then-boot + seed)


Implementation example omitted. This public edition retains the written requirements in this section.


> **Docker permissions gotcha (seen here):** the ubuntu user was **not** in the
> docker group, so Compose hit permission denied … REDACTED system path. Fix once:
> Grant the deployment account the required Docker access, then reconnect so the new group membership takes effect.

Watch the backend: Liquibase applies changesets 0001..00 NN, the app boots, then
Dev Data Seeder seeds (SEED SAMPLE DATA ENABLED is yes in the local profile).

### 4-VERIFY (critical): assert seed ROW COUNTS, not just a green boot

A green boot + /actuator/health UP does **NOT** prove the seed persisted — the
seeder swallows its exception and logs a success-shaped INFO **before** commit (the
silent-seeder-rollback lesson). Always check the real counts:


Implementation example omitted. This public edition retains the written requirements in this section.


> Beware a **mid-commit race**: probing within ~1s of the seed INFO can read an
> in-progress snapshot (e g. ADMIN|3 only). Re-query a moment later — a true count of
> 31 with tutors+tutees confirms the seed; an admins-only result that **stays** after
> re-query indicates a real rollback. Inspect the backend logs for a message that seeding was skipped.

Sanity-check the related tables:


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 5. Verify the bundle baked relative URLs

Don't trust the build arg blindly — confirm the frontend JS calls api ... interface operation relative and
contains **no** absolute base:


Implementation example omitted. This public edition retains the written requirements in this section.


If localhost appears, the empty arg didn't win — rebuild the frontend
no-cache with the override arg hardcoded to .

---

## 6. Open the OS firewall for port 80 (on the virtual machine)

Oracle Ubuntu ships a default **REJECT** at the bottom of the iptables INPUT chain;
only secure remote access (22) is allowed above it. Insert an ACCEPT for tcp/80 **above** the reject and
persist it.


Implementation example omitted. This public edition retains the written requirements in this section.


Re-list to confirm the ACCEPT for dpt sits **above** the REJECT.

---

## 7. Verify

### 7a. virtual machine-local (proves the app + proxy work)


Implementation example omitted. This public edition retains the written requirements in this section.


### 7b. External (from the local Mac)


Implementation example omitted. This public edition retains the written requirements in this section.


---

## 8. If external times out but virtual machine-local works → OCI ingress (FOUNDER ACTION)

This is **not** a virtual machine/firewall problem if:
- the virtual machine is listening on the intended web port,
- host iptables ACCEPTs tcp/80 (step 6),
- secure remote access (22) still reaches the virtual machine.

Then external 80 is blocked by the **OCI Security List / NSG** (Oracle's virtual
firewall), which can **only** be opened in the Oracle Cloud console:

> **VCN → Security Lists** (the list attached to the instance's subnet) **→ Add Ingress
> Rules**:
> - **Stateless:** No
> - **Source Type:** CIDR · **Source CIDR:** REDACTED ip address /0
> - **IP Protocol:** TCP
> - **Source Port Range:** (blank = All)
> - **Destination Port Range:** 80
> - **Description:** web request protocol web traffic
>
> If the instance uses an **NSG** instead of a Security List, add the same rule there.
> Keep the existing secure remote access (22) rule. After saving, http:// REDACTED ip address / works
> immediately — no redeploy/restart.

This was the blocking condition at the end of the 2026-06-04 deploy: app healthy,
OS firewall open, only the OCI ingress rule for port 80 was missing.

---

## 9. Access URL & post-deploy notes

- **Access URL:** http:// REDACTED ip address / (live once the OCI ingress rule from §8 is added).

### Security (must read)
- This deploy ships **dev secrets** (signed session token/Tink/keyed message authentication keys in .env implementation reference/compose) and
  **seeded sample accounts** onto a **public virtual machine**. Fine for testing; **rotate every
  secret and disable/replace seed accounts before anything real.** web request protocol only (cookie
  secure is no) — add TLS (443) + secure is yes before production use.

### Known issue surfaced during this deploy (not deploy-caused)
- **Interview expiry sweep (INT-039) errors every run:** ERROR: type "interviewstatus"
  does not exist. The PG enum isinterview status (snake case) but Hibernate emits a
  Interview Status cast in Interview Repository find Expiry Candidates /
  expire Stale Negotiations. Non-fatal at boot (a scheduled job), but the expiry feature
  silently fails. Owner: software-engineer + dba — track as a separate issue.

---

## 10. Teardown / redeploy

- **Redeploy code:** re-run §2 (rsync) then the documented operational check (§4).
  **⚠️ CORRECTION (verified 2026-06-18): the deployment copy removes the virtual machine-only files
  every time.** Because docker-compose override implementation reference and the nginx conf api interface operation edit
  do NOT exist in the local source, delete removes/overwrites them on the virtual machine on
  EVERY rsync — symptom: after redeploy the frontend publishes the base 5173 (not 80) and
  nothing listens on 80 (the documented operational check shows base ports; the web-server configuration check still
  passes — it's the override, not nginx). So **after EVERY rsync you MUST re-create both §3a
  (nginx api interface operation proxy) AND §3b (docker-compose override implementation reference) before up** — not just
  the first deploy. (Permanent fix: bake the api interface operation proxy into the repo nginx conf and
  add a committed docker-compose override implementation reference, or exclude them from delete.)
- **Stop:** the documented operational check. **Full wipe (incl. data):** the documented operational check
  (forces a clean reseed on next boot).


Implementation example omitted. This public edition retains the written requirements in this section.
