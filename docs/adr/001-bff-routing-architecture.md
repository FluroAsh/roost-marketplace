# 001 — Route the whole API through a single Lambda-lith behind an API Gateway proxy

- Status: accepted
- Date: 2026-07-04

## Context

- The BFF routing solution must be performant, and given the size of the team (1) we should not over-engineer for least-privilege as there is no team-level separation required.
- Local development experience matters disproportionately at solo scale — every hour spent on AWS emulation is an hour not spent building.
- We evaluated separating each route into its own isolated Lambda and offloading routing to the API Gateway layer — this would contribute significant overhead to our Terraform/IAM configuration for no benefit at a team size of one.

## Decision

We will use a single API Gateway proxy route (`ANY /{proxy+}`) with a Hono app deployed as a single Lambda ("Lambda-lith"); route dispatch happens inside the Hono router rather than at the API Gateway layer.

## Consequences

- We lose _per-route_ API Gateway configuration: policies, throttling overrides, observability and permissions can no longer be set on individual routes at the gateway. Stage-level throttling and the JWT authorizer still apply API-wide at the gateway; anything finer-grained (per-route rate limits, per-route metrics) must be handled inside the Hono application via middleware and structured logging.
- The blast radius becomes the entire Hono application, as opposed to single domain-based API Gateway/Lambda-driven endpoints — one bad deployment breaks _all_ endpoints until fixed.
- What we gain is the ability to run the local web server directly without SAM/emulators — the same Hono app runs locally via `pnpm dev` and deploys bundled into Lambda.
- **Upgrade trigger:** not traffic (a lith scales horizontally fine) — the split to domain-scoped Lambdas is forced by team count and isolation requirements: per-domain IAM boundaries, independent deploy cadences, or per-domain tuning needs. Until a second team or a hard isolation requirement exists, the lith holds.
