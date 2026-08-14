---
name: security-review
description: >
  Adversarial security review of a change before it ships. Answers one question with
  evidence: what can an untrusted caller reach that they should not? Covers privilege
  reachability across trust boundaries, authorization vs authentication, caller-supplied
  identity and values, secret handling, data isolation, injection/SSRF/traversal, and abuse
  of the operation itself. Stack-agnostic — the boundary may be a browser, a mobile client,
  a public API, a webhook, a queue, a CLI, a third-party integration, or a tool exposed to an
  agent. Mandatory as the final gate of any plan that adds a request, an entry point, a data
  read/write, a permission, or secret handling. Use before merging, not after.
argument-hint: "[diff | plan path | 'branch'] "
allowed-tools: Read, Grep, Glob, Bash, Agent
---

# Security Review

A **final, adversarial** pass over a change. Not a lint, not a style review: the question is
always **"what can an untrusted caller reach that they should not?"** — and every answer is
backed by a command and its output, never by a rule someone wrote.

> **The shape of the bug this exists to catch.** A privilege-elevating database function shipped
> with execute permission granted to the ordinary logged-in role. The function bypassed row-level
> isolation by design, so any authenticated user could invoke it directly and read — and modify —
> other tenants' data. Every individual piece was "correct": the function was reviewed, the
> endpoint had authentication, the table had isolation policies. Nobody asked what a caller on the
> other side of the trust boundary could invoke without going through the endpoint at all.
>
> That story is one instance, not the category. The same defect appears as a mobile client holding
> an admin token, a queue any producer can write to, an internal service that trusts a header
> anyone can set, or a tool handed to an agent with wider scope than the conversation it serves.

## When this gate is MANDATORY

Any change that adds or modifies:

- an **entry point** — endpoint, route, webhook, queue consumer, scheduled job, CLI command,
  RPC, agent-callable tool, or anything else that executes on someone else's input;
- an **outbound request** to a service you do not control;
- a **data read or write** — query, stored procedure, migration, trigger, view, cache, file, blob;
- a **permission**: a role, a grant, a scope, a token audience, a flag that gates access, an
  allowlist;
- **secret handling**: credentials, keys, signing, rotation, environment configuration.

Pure presentation changes and copy/config edits with no new data access or new caller are exempt.
When in doubt, run it — it costs minutes.

## The rule that makes this worth running

**A written rule is never evidence of system state.** "The privileged credential is only used in
the server layer" bounds where it is *allowed*; it does not prove every file obeys it, and it says
nothing about what is *reachable*. Prove the negative with a command, and make the command find a
positive case you know exists before trusting an empty result.

## Procedure

### 1. Map the trust boundaries this change touches

Get the actual diff — never review from memory or from the plan alone.

```
git diff origin/<base>...HEAD --stat
git diff origin/<base>...HEAD
```

Then write down, explicitly, two lists:

1. **Entry points added or modified**, and for each: **who can call it** (anonymous · any
   authenticated user · a specific role · another service · a third party holding a token) and
   **what it can do** once called.
2. **Privileged capabilities added or modified**: anything that runs with more authority than its
   caller — elevated DB functions, admin credentials, signing keys, impersonation, scope-widening
   tokens, unrestricted internal endpoints.

The review is the cross-product of those lists: for each capability, which of those callers can
reach it, directly or transitively? Anything not on the lists is out of scope for this pass, and
say so.

### 2. Run the seven checks

Each check states **what to prove**, not what to read. Record the command and its output.

#### C1 · Privilege reachability across the boundary

The highest-severity class. A privileged capability reachable by an untrusted caller makes every
other control decorative, because the attacker simply skips the layer that enforces them.

For each privileged capability, prove that **every** path to it crosses a check:

- **Direct invocation.** Can the caller invoke the capability without going through your
  intended entry point? This is the one people miss: the endpoint is not the only door. Elevated
  database functions callable by the client role, storage buckets addressable by URL, internal
  endpoints reachable from outside the network, queues any producer can publish to, admin methods
  exposed on a shared object.
- **Credential placement.** A credential with more authority than the caller must never live
  where the caller can obtain it: shipped in a client bundle, embedded in a mobile binary,
  returned in a response, readable from a config the caller controls. Grep the **built artifact**,
  not only the source.
- **Delegation and scope.** When your code acts on behalf of a caller, confirm the downstream
  call carries the *caller's* authority, not the service's. Anything that runs as the owner rather
  than the invoker re-checks nothing.

Worked example — an elevated function in a Postgres-based stack, where the client role is granted
permissions explicitly and revoking from `PUBLIC` alone is not enough:

```sql
SELECT p.proname,
       has_function_privilege('anon',          p.oid, 'EXECUTE') AS anon,
       has_function_privilege('authenticated', p.oid, 'EXECUTE') AS auth
FROM pg_proc p JOIN pg_namespace n ON n.oid = p.pronamespace
WHERE n.nspname = 'public' AND p.prosecdef;
```

The equivalent in another stack is the same question with different nouns: which principals hold
the permission, checked against the live system, not the code that was supposed to set it.

#### C2 · Authorization, not just authentication

"Is there a valid session/token?" is not authorization. For every entry point, prove that caller A
cannot reach caller B's object: does the handler check **ownership or tenancy** against the
specific identifier it was given, or only that someone is logged in? Guessable identifiers make
this exploitable in a loop. Confirm role and scope checks are **fail-closed** (unknown ⇒ deny) and
evaluated at the granularity that matters — an allowlist of *fields* does not restrict *keys
nested inside* one of them.

#### C3 · Never trust the caller for identity or value

Tenant, user, role, scope, price, quantity, status: if it arrives in the request and is used
without re-derivation from the authenticated principal, it is forgeable. Derive server-side. The
same applies to values a caller can influence indirectly — a header set by a proxy anyone can
reach, a field an earlier untrusted step wrote.

#### C4 · Secrets

No secret in an artifact the caller receives, in logs, in error messages, in a URL or query
string, or in a cache key. Check the diff for new logging near credentials. Confirm any new secret
exists in every environment that reads it — a path that silently falls open when it is missing is
a finding, not a convenience.

#### C5 · Data isolation

New data store or collection ⇒ isolation enabled **and** a policy that scopes it. Isolation with
no policy denies everything; a policy with isolation disabled enforces nothing. Verify both, and
check that writes are restricted separately from reads — a derived or cached dataset usually wants
"no writes from any caller". Confirm that aggregates, exports, search indexes and backups inherit
the same scoping; they are the usual leak.

#### C6 · Injection, SSRF, traversal, deserialization

Queries or commands built from caller input need parameterization, and identifiers need an
allowlist — interpolation is not escaping. Any outbound request to a caller-influenced destination
needs a destination allowlist evaluated **before** the request. Paths and filenames need
normalization checks. Deserializing caller-supplied structures needs a schema, not trust.

#### C7 · Abuse of the operation itself

Rate limiting on anything expensive or enumerable; idempotency on anything that moves money,
sends a message, or provisions a resource; replay protection on anything signed (signature **and**
freshness). Ask what happens when the operation is called a thousand times, and when it is called
twice with the same input.

### 3. Verify, do not assert

For each finding, produce the evidence. For each *cleared* check, produce the command that cleared
it. A check with no command behind it did not happen — say so rather than implying it passed.

Prefer a live probe when the surface allows one: call the entry point or the capability as the
untrusted principal and show the denial. A refusal you observed beats a permission you read.

### 4. Report

```
## Security review — <change>

Boundaries reviewed: <entry points → who can call them>
Privileged capabilities: <what runs with more authority than its caller>

| # | Check | Verdict | Evidence |
|---|-------|---------|----------|
| C1 | Privilege reachability | PASS/FAIL/N-A | <command + result> |
| ... |

### Findings
- [CRITICAL] <the attacker's path, concretely: which principal, through which surface, obtaining what>
- [HIGH] ...

### Not covered
<what this pass did not check, and why>
```

Severity is about **reachability and blast radius**, not effort to fix. Reachable by an untrusted
caller and crossing an isolation boundary is CRITICAL, however small the patch.

## Rules

- **Never mark a check PASS from reading code or documentation alone** when a command could have
  proven it. Cite the command.
- **A finding names the attacker's path**, not the smell: "any authenticated user invokes
  capability `x` directly and modifies another tenant's rows" — not "this function looks unsafe".
- **Report an empty result as a limitation, not as proof**, unless the search was validated
  against a known positive.
- **Read-only.** This skill finds and reports; it does not patch. Fixes are a separate, reviewable
  change.
- **Do not skip the gate because the change is small.** The incident that motivated this was one
  permission grant.
