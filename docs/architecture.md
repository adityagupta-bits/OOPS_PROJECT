# Architecture

BITSolve is a modular campus IT service desk with Student, Technician and Administrator portals. It uses React 19/Vite, Java 25/Spring Boot 4.1.2 and PostgreSQL 18. It is not a microservice system and does not use a machine-learning classifier or external message broker.

| Layer | Implementation | Responsibility |
|---|---|---|
| Browser | React and shared CSS | Role-specific forms, incident details, conversations and countdown display |
| HTTP security | Spring Security | Session authentication, CSRF protection, enabled-account and role checks |
| API | Spring MVC controllers and DTOs | Request validation, authenticated identity and structured responses |
| Application | Domain services | Transactions, workflow permissions, classification, allocation and SLA rules |
| Persistence | Spring Data JPA, Hibernate and selected JDBC | Entities, queries, locking and durable settings |
| Database | PostgreSQL, Flyway V1–V19 | Relationships, constraints and versioned schema |
| Background | Spring scheduled workers | Pending assignments, SLA monitoring, escalation and optional email delivery |

The production JAR includes `frontend/dist` and serves React with `/api` on port 8080. During development, Vite on 5173 proxies `/api` to the backend. PostgreSQL normally uses 5432. Ports are configurable.

[Architecture diagram](diagrams/architecture.md)

## Modules

- **Authentication:** `AuthController`, `AuthService`, `SecurityConfig`, `EnabledAccountFilter`, `PasswordRecovery` and the `User` hierarchy.
- **Lifecycle:** `IncidentController`, `IncidentService`, `Incident`, `IncidentHistory` and `InvestigationNote`.
- **Catalog/classification:** `CatalogService`, `CatalogEntry`, `ClassificationSettings` and `IncidentClassificationService`. Service/category identities are string-backed value types, allowing custom catalog entries without new Java enum constants.
- **Priority:** `PriorityService` selects weighted, multiplicative, matrix or rule-based `PriorityStrategy` from persisted settings.
- **Assignment:** `AssignmentService`, `WorkloadService`, skill/load strategies and `RandomAvailableAssignment` for general support; a worker retries pending allocation.
- **SLA/escalation:** `SlaPolicies` captures targets/deadlines; `SlaMonitor`, `ScheduledEvents` and `EscalationService` operate without a browser session.
- **Communication:** `IncidentConversation` separates reporter-visible messages from incident-scoped private staff messages and persists read positions.
- **Notification/audit:** `EventBus` invokes `IncidentObserver` implementations; delivery and audit records are persisted.

## Security and consistency

Passwords are BCrypt hashes. Authentication uses an HTTP-only session cookie; mutations require CSRF protection. Public registration creates students only. Services enforce ownership and team membership in addition to role checks. Hidden frontend controls are not the authorization boundary.

Internal staff chat is restricted to administrators and the incident's authorized technicians, not every staff member. Student-visible conversation is separate. Disabled users lose access on subsequent requests.

Workflow changes are transactional. EventBus dispatch is in-process; observer failures propagate so the transaction can roll back. It is not an asynchronous broker. Scheduled-event keys persist warning/escalation claims across restarts. Allocation checks active state, availability, skill and workload capacity.

Configuration resides in PostgreSQL. Incidents retain captured classification/priority evidence and SLA deadlines; priority overrides do not silently restart their clocks. Transfers and collaborations are separate audited operations.

## Scope

Classification uses deterministic keywords and rules. It does not guarantee recognition of every description. Ambiguity or missing configuration enters review; lack of an eligible technician can leave a classified incident pending assignment. SLA countdowns show deadlines, not predicted repair time. SMTP requires private sender configuration and was not tested for this submission.

See [workflows](workflows.md), [database](database.md), [OOP design](object-oriented-design.md) and [verification](VERIFICATION.md).
