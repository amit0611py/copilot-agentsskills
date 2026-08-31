# Copilot instructions — Payment project

## Repository layout
- `frontend/` — Angular application. Currently v13, being upgraded to v21.
- `backend/` — API and HTML shell serving. CSP work happens here.
- `.github/` — workflows, agent profiles, instructions.

## Agents
- `@angular-upgrade` — owns `frontend/`. One Angular major version per PR.
- `@csp-hardening` — owns `backend/security/`. Nonce-based CSP, Report-Only first.

An agent must not edit outside its directory. Cross-cutting changes go in a separate PR
with both owners as reviewers.

## Hard guardrails (apply to every agent and every session)

1. This is a payment system. Any change to money movement, amount calculation, currency
   handling, idempotency keys, retry logic, or webhook verification is out of scope for
   the upgrade and CSP work. If a migration touches those files, stop and ask.
2. No opportunistic refactoring. If it is not required to make the build, tests, or lint
   pass, it does not belong in the PR.
3. No new dependencies without justification in the PR body.
4. Never commit secrets, nonces, keys, or `.env` values.
5. `--force` is forbidden. `--legacy-peer-deps` requires an explicit note naming the
   package that needed it.
6. Never disable a test to make a hop green. A failing test is a finding, not an obstacle.
7. Never suppress a TypeScript error with `any`, `as unknown as`, or `@ts-ignore` to get
   past a version bump. Fix it or stop.

## Definition of done for any PR here
Production build passes, unit tests pass, lint passes, visual regression diff is clean or
justified image-by-image, and the payment smoke flows in `docs/upgrade/runbook.md` were
exercised manually with screenshots attached.

## Escalation
Open an issue rather than improvising when: a dependency has no compatible version, a
visual diff cannot be explained, a CSP violation comes from a third-party payment SDK, or
a change would alter observable application behaviour.
