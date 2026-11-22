# Core database relationships

```mermaid
erDiagram
    app_users ||--o{ incidents : reports
    app_users o|--o{ incidents : primary_technician
    app_users ||--o{ technician_skills : has
    app_users ||--o{ incident_collaborators : assists
    incidents ||--o{ incident_collaborators : has
    incidents ||--o{ incident_history : records
    incidents ||--o{ investigation_notes : contains
    app_users ||--o{ investigation_notes : authors
    incidents ||--o| incident_feedback : receives
    app_users ||--o{ incident_feedback : submits
    incidents ||--o{ transfer_requests : has
    incidents ||--o{ collaboration_requests : has
    app_users ||--o{ password_reset_tokens : owns
    app_users {
        bigint id PK
        varchar role
        varchar email UK
        varchar password_hash
        boolean enabled
        varchar availability
        integer max_workload
    }
    incidents {
        bigint id PK
        bigint reporter_id FK
        bigint assigned_technician_id FK
        varchar affected_service
        varchar category
        varchar priority
        varchar status
        timestamptz response_deadline
        timestamptz resolution_deadline
    }
    incident_collaborators {
        bigint id PK
        bigint incident_id FK
        bigint technician_id FK
        varchar responsibility
    }
    catalog_entries {
        varchar id PK
        varchar kind
        varchar code
        varchar label
        boolean enabled
        varchar criticality
        varchar keywords
    }
```

Selected core relationships only. Transfer/collaboration records also reference requesting, proposed and reviewing users. `catalog_entries` is intentionally separate: incident service/category codes are logical references, not physical foreign keys to its ID. Configuration, notification, read-receipt, audit, scheduled-event and Flyway tables are described in [Database](../database.md); not every stored identifier is a foreign key.
