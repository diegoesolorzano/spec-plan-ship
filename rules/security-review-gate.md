# Security Review Gate

Every plan ends with a **final security review** as a fixed, non-optional task, and the
`security-review` skill is invoked to run it. Detail (the seven checks, the evidence
protocol, the report format) → that skill.

- **CRITICAL trigger — the gate is mandatory** when the change adds or modifies an entry point
  (endpoint, webhook, queue consumer, scheduled job, CLI command, RPC, agent-callable tool), an
  outbound request to a service you do not control, a data read/write, a permission
  (role/grant/scope/token audience/access flag), or secret handling. Pure presentation and
  copy/config changes with no new data access and no new caller are exempt.
- **Highest-priority check: privilege reachability across the trust boundary.** A capability that
  runs with more authority than its caller — an elevated data function, an admin credential, a
  signing key, a scope-widening token — must be unreachable from every untrusted caller, by every
  path, not only through the intended entry point. The entry point is never the only door.
- **Evidence, never assertion.** A check is PASS only with the command and its output; a rule
  saying "X is only allowed in Y" bounds where X is *permitted* and never proves what is
  *reachable*. An empty search is a limitation, not proof of absence.
- **The gate runs before merge, and it is read-only** — findings become a separate, reviewable
  fix. It does not replace the cross-model review; it is the last thing that runs.
