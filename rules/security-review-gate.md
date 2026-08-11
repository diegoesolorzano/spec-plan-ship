# Security Review Gate

Every plan ends with a **final security review** as a fixed, non-optional task, and the
`security-review` skill is invoked to run it. Detail (the seven checks, the evidence
protocol, the report format) → that skill.

- **CRITICAL trigger — the gate is mandatory** when the change adds or modifies a fetch/HTTP
  call, an API route or webhook, a DB read/write (query, RPC, migration, trigger, view,
  policy), a permission/grant/role/access flag, or secret handling. Pure-UI and copy/config
  changes with no new data access are exempt.
- **Highest-priority check: what can the BROWSER reach?** A privileged credential or a
  privilege-elevating DB function callable by `anon`/`authenticated` makes every other control
  decorative. Revoking from `PUBLIC` alone is not enough — platforms grant the client roles
  explicitly.
- **Evidence, never assertion.** A check is PASS only with the command and its output; a rule
  that says "X is only allowed in Y" bounds where X is *permitted* and never proves what is
  *reachable*. An empty search is a limitation, not proof of absence.
- **The gate runs before merge, and it is read-only** — findings become a separate, reviewable
  fix. It does not replace the cross-model review; it is the last thing that runs.
