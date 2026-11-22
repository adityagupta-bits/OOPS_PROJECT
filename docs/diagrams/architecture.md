# System architecture

```mermaid
flowchart TB
    Users[Student / Technician / Administrator] --> UI[React role-based portals]
    UI -->|Session cookie and CSRF token| Security[Spring Security]
    Security --> API[Spring MVC controllers]
    API --> Domain[Transactional domain services]
    Domain --> Rules[Catalog matching and priority strategies]
    Domain --> Assignment[Eligibility and assignment strategies]
    Domain --> Repo[JPA repositories and JDBC]
    Repo --> DB[(PostgreSQL)]
    Domain --> Events[In-process EventBus]
    Events --> Observers[Audit and notification observers]
    Observers --> Repo
    Workers[Scheduled workers] --> Assignment
    Workers --> SLA[SLA monitor and escalation]
    SLA --> Events
    SLA --> Repo
    Workers --> Mail[Optional email outbox delivery]
    Mail --> DB
    Mail --> SMTP[Gmail SMTP when configured]
    Flyway[Flyway V1-V19] --> DB
```

The executable JAR serves the built frontend and API together. Development uses Vite with an API proxy. EventBus is in-process, not Kafka/RabbitMQ. Workers run while the backend is running; no user login is required.
