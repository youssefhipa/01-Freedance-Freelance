# My Contributions — Youssef Hipa

Freedance is a freelance marketplace built by a team of about 20 engineers as a
microservices system: 5 Spring Boot services behind a Spring Cloud Gateway,
RabbitMQ sagas, Kubernetes, and a polyglot data layer (PostgreSQL, MongoDB,
Redis, Elasticsearch, Neo4j, Cassandra).

**My role:** owner of **contract-service**, and author of the **Next.js frontend**.

| | |
|---|---|
| Commits | 20 (contract-service, cross-service saga work, frontend) |
| Lines added | ~17,300 across 160 files |
| Stack | Java 25, Spring Boot 4, Spring Data JPA, OpenFeign, RabbitMQ, Cassandra, JUnit 5, Next.js 14, TypeScript, TanStack Query, Tailwind |

---

## 1. Contract service (owner)

The contract service manages the lifecycle of a contract between a client and
a freelancer once a proposal is accepted: creation, status changes, milestones,
and reporting.

### Features I built (S4-F7 … S4-F12)

| Feature | What it does | Endpoint |
|---|---|---|
| Data retention | Purge contract data older than N days | `DELETE /api/contracts/purge?olderThanDays=` |
| Freelancer performance | Per-freelancer completion and earnings summary | `GET /api/contracts/freelancer/{id}/summary` |
| Stalled contracts | Find active contracts with no recent progress | `GET /api/contracts/stalled` |
| Milestone tracking | Record milestone events (time-series in Cassandra) | `POST /api/contracts/{id}/milestones/track` |
| Milestone timeline | Ordered milestone history for a contract | `GET /api/contracts/{id}/milestones/timeline` |
| JWT-secured access | Token parsing and ownership checks on contract operations | `JwtTokenService` |

Each feature shipped with controller tests and service or integration tests
(for example `ContractServiceF789Test`, `ContractF12IntegrationTest`).

### M3 refactor: from shared reads to service boundaries

- Replaced direct data access with **OpenFeign clients** (`UserServiceClient`,
  `JobServiceClient`) so contract-service reads other services only through their APIs.
- Added internal endpoints that other services call during the saga:
  `/user/{id}/summary`, `/user/{id}/active-count`, `/user/{id}/completed-count`,
  `/job/{id}/active-count`, `/proposal/{id}/active`.
- Added **`ContractEventPublisher`** (`ContractCreated`, `ContractStatusChanged`
  events on RabbitMQ) and **`ContractSagaConsumer`** to take part in the
  proposal → contract → wallet saga.

### Saga reliability (cross-service)

- Added the **saga pre-check failure test**. It proves that when a pre-check
  fails, the proposal service publishes the failure event and no contract is created.
- Updated `ProposalEventPublisher` / `ProposalService` in proposal-service to
  support it.
- Fixed an H2 JDBC naming fallback that made saga tests behave differently from
  PostgreSQL, and aligned the saga test assertions.
- Resolved a duplicate Spring Security filter chain introduced by a main-branch merge.

---

## 2. Frontend (author)

I built the full **Next.js 14** frontend (58 files, ~13k lines) and connected it
to the live microservices through the API gateway.

- **Auth:** login and register with JWT.
- **Role dashboards:** client, freelancer, and admin views.
- **Marketplace flows:** jobs, proposals, contracts (list + detail with
  milestones), wallet / payouts, users, settings.
- **Engineering views:** a saga page that visualises the distributed transaction
  steps, and an observability page.
- **Stack:** App Router, TypeScript, TanStack Query, react-hook-form, Tailwind,
  Recharts, Playwright for the recorded live demo.

---

## What I learned

- Designing service boundaries, and the cost of crossing them (Feign calls and
  failure handling).
- Orchestrating distributed transactions with **sagas** and writing tests for
  failure paths, not just happy paths.
- Working in a large team: strict branch and commit conventions, pre-push test
  gates, and frequent merges into a shared `main`.

> Original repository: [Freelance-acl/01-Freedance-Freelance](https://github.com/Freelance-acl/01-Freedance-Freelance)
