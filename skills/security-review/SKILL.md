---
name: security-review
description: >
  Adversarial security review of a change before it ships. Answers one question with
  evidence: what can an ATTACKER do with the surface this change adds? Covers privilege
  bypass (service-role/admin keys reachable from the client, SECURITY DEFINER RPCs granted
  to anon/authenticated), missing authorization (authenticated but not authorized, IDOR),
  client-trusted input, secret exposure, RLS gaps, SSRF and injection. Mandatory as the
  final gate of any plan that adds a fetch, an API route, a DB read/write, an RPC, a
  migration, or a new permission. Use before merging, not after.
argument-hint: "[diff | plan path | 'branch'] "
allowed-tools: Read, Grep, Glob, Bash, Agent
---

# Security Review

A **final, adversarial** pass over a change. Not a lint, not a style review: the question is
always **"what can an attacker reach that they should not?"** — and every answer is backed by a
command and its output, never by a rule someone wrote.

> **Why this exists.** A `SECURITY DEFINER` function shipped with `EXECUTE` granted to the
> `authenticated` role. The function bypassed row-level security by design, so any logged-in
> user could call it straight from the browser console and read — and modify — other tenants'
> data. Every individual piece was "correct": the function was reviewed, the endpoint had auth,
> the table had RLS. Nobody asked what the browser could call directly.

## When this gate is MANDATORY

Any change that adds or modifies:

- a **fetch/HTTP call**, an API route, a webhook, or a public endpoint;
- a **database read or write** — query, RPC, migration, trigger, view, policy;
- a **permission**: a role, a grant, a flag that gates access, an allowlist;
- **secret handling**: env vars, tokens, signing, key rotation.

Trivial copy/config edits and pure-UI changes with no new data access are exempt. When in
doubt, run it — it costs minutes.

## The rule that makes this worth running

**A written rule is never evidence of system state.** "Service role is only used in `app/api/`"
bounds where it is *allowed*; it does not prove that every file obeys it, and it says nothing
about what is *reachable*. Prove the negative with a command, and make the command find a
positive case you know exists before trusting an empty result.

## Procedure

### 1. Establish the attack surface

Get the actual diff — never review from memory or from the plan alone.

```bash
git diff origin/<base>...HEAD --stat
git diff origin/<base>...HEAD -- '*.sql' '*.ts' '*.tsx' '*.js'
```

List, explicitly: every new endpoint, every new DB entrypoint, every new grant, every new
client-side call. That list is what you review; anything not on it is out of scope for this pass.

### 2. Run the seven checks

Each check states **what to prove**, not what to read. Record the command and its output.

#### C1 · Privilege bypass — what can the BROWSER call directly?

The highest-severity class, and the one that caused the incident above. A privileged credential
or a privilege-elevating function that a client can reach makes every other control decorative.

- **Elevated DB functions:** every `SECURITY DEFINER` function must have `EXECUTE` revoked from
  `PUBLIC`, `anon` and `authenticated`, and granted only to the backend role. Revoking from
  `PUBLIC` alone is **not enough** when the platform grants the client roles explicitly.
  Verify against the live ACL, not the migration text:
  ```sql
  SELECT p.proname,
         has_function_privilege('anon',          p.oid, 'EXECUTE') AS anon,
         has_function_privilege('authenticated', p.oid, 'EXECUTE') AS auth
  FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
  WHERE n.nspname = 'public' AND p.prosecdef;
  ```
  Any `true` in a function that bypasses row-level security is a **CRITICAL** finding.
- **Privileged keys:** the service-role/admin key must never be importable from client code, and
  never inlined into the browser bundle. Grep the built output, not just the source.
- **Views:** a view over tenant-scoped tables must run as the *caller* (e.g.
  `security_invoker = true`), or it silently returns every tenant's rows. `CREATE OR REPLACE`
  does not always preserve that option — assert it after the fact.

#### C2 · Authorization, not just authentication

"Is there a session?" is not authorization. For every new endpoint, prove that user A cannot
reach user B's object: does the handler check **ownership/tenant** on the specific id it was
given, or only that someone is logged in? Sequential/guessable ids make this exploitable in a
loop. Also confirm role checks are **fail-closed** (unknown role ⇒ deny) and evaluated at the
granularity that matters — an allowlist of *columns* does not restrict *keys inside a JSON
column*.

#### C3 · Never trust the client for identity or value

Tenant id, user id, role, price, quantity, status: if it arrives in the request body or a query
param and is used without re-derivation from the session, it is forgeable. Derive server-side.

#### C4 · Secrets

No secret in a client-exposed env var (anything the framework inlines into the bundle), in logs,
in error messages returned to the user, or in a URL. Check the diff for new logging around
credentials. Confirm any new secret was set in every environment it is read from — a
fail-open path around a missing secret is a finding.

#### C5 · Data-layer isolation

New table ⇒ row-level security enabled **and** a policy that scopes it. RLS without a policy
denies everything; a policy without RLS enforces nothing. Verify the enabled flag and the
policy list, and check that writes are restricted separately from reads — a derived table
usually wants "nobody writes from the client".

#### C6 · Injection, SSRF, and traversal

Dynamic SQL built from user input needs an allowlist of identifiers, never interpolation.
Any `fetch` to a URL influenced by user input needs a destination allowlist before the request.
Filenames and paths from input need normalization checks.

#### C7 · Abuse of the operation itself

Rate limiting on anything expensive or enumerable; idempotency on anything that moves money or
sends a message; replay protection on webhooks (signature + timestamp). Ask what happens when
the operation is called a thousand times, and when it is called twice with the same input.

### 3. Verify, do not assert

For each finding, produce the evidence. For each *cleared* check, produce the command that
cleared it. A check with no command behind it did not happen — say so rather than implying it
passed.

Prefer a live probe when the surface allows one: call the endpoint or the RPC with a client-role
token and show the `403`/permission error. A denial you observed beats a grant you read.

### 4. Report

```
## Security review — <change>

Surface reviewed: <endpoints / RPCs / migrations / grants>

| # | Check | Verdict | Evidence |
|---|-------|---------|----------|
| C1 | Privilege bypass | PASS/FAIL/N-A | <command + result> |
| ... |

### Findings
- [CRITICAL] <what an attacker does, concretely: who, through which surface, obtaining what>
- [HIGH] ...

### Not covered
<what this pass did not check, and why>
```

Severity is about **reachability and blast radius**, not effort to fix. Reachable from an
unauthenticated browser and crossing a tenant boundary is CRITICAL, however small the patch.

## Rules

- **Never mark a check PASS from reading code or documentation alone** when a command could
  have proven it. Cite the command.
- **A finding names the attacker's path**, not the smell: "a logged-in user of tenant A calls
  RPC `x` from the browser and updates tenant B's rows" — not "this function looks unsafe".
- **Report an empty result as a limitation, not as proof**, unless the search was validated
  against a known positive.
- **Read-only.** This skill finds and reports; it does not patch. Fixes are a separate,
  reviewable change.
- **Do not skip the gate because the change is small.** The incident that motivated this was
  one `GRANT` line.
