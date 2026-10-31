# ADR-0001: Environments and promotion

- Status: Proposed
- Date: 2026-10-31
- Deciders: GchengC

## Context

gc-platform provides a base platform (Spring Boot starters, an Angular library and a project template) that downstream projects consume. We need a shared definition of the environments a change goes through and the rules for moving it from one to the next, so that every project built on the platform behaves the same way.

## Decision

### Environments

| Environment | Purpose | Trigger |
|-------------|---------|---------|
| Local | Development and fast feedback on a developer machine | Manual |
| CI | Build, lint, unit and integration tests on a clean Linux runner | Every push and pull request |
| Staging | Production-like environment for validation of release candidates | Merge to `main` |
| Production | Live environment | Approved release tag |

### Promotion rules

1. Every change starts on a branch `feat/<issue>-<slug>` or `fix/<issue>-<slug>` (or `docs/<issue>-<slug>`) linked to an issue.
2. A pull request must pass CI before it can be merged into `main`. `main` is only updated through reviewed and approved pull requests.
3. Merging to `main` produces a versioned, immutable build artifact. The same artifact is promoted through the environments; it is never rebuilt per environment.
4. The artifact is deployed to staging automatically and validated there.
5. Production receives the artifact that was validated in staging, identified by a SemVer tag (`vMAJOR.MINOR.PATCH`), after explicit approval by the maintainer.
6. Environment differences are expressed only through external configuration and secrets, never through code changes.

### Rollback

Rolling back means redeploying the previous tagged artifact. Database migrations must be backward compatible with the previous release so that a rollback does not require a schema rollback.

## Consequences

- Staging and production run identical artifacts, which reduces environment-specific defects.
- Releases are traceable to a tag, a pull request and an issue.
- Production deployments require a manual approval step, trading some speed for safety.
- Backward-compatible migrations add some design overhead to schema changes.
