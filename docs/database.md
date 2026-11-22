# Database

PostgreSQL is the system of record. Hibernate validates the schema; Flyway owns evolution. All V1–V19 migration files are retained unchanged. The export includes migration checksums and sequence values.

| Area | Tables |
|---|---|
| Accounts | `app_users`, `technician_skills`, `password_reset_tokens` |
| Catalog | `catalog_entries`, `problem_templates` |
| Configuration | `classification_settings`, `priority_settings`, `assignment_settings`, `sla_policies` |
| Incident and team | `incidents`, `incident_collaborators`, `transfer_requests`, `collaboration_requests` |
| Work records | `incident_history`, `investigation_notes`, `incident_feedback` |
| Delivery | `notification_deliveries`, `incident_message_reads` |
| Audit/monitoring | `audit_events`, `scheduled_events` |
| Schema version | `flyway_schema_history` |

User subclasses share `app_users` with a role discriminator. Incidents reference a reporter and optionally a primary technician; collaborators form a separate relationship. Service/category codes are logical catalog references stored as strings, not foreign keys to catalog numeric IDs. Historical values remain readable after catalog changes.

[Core ER diagram](diagrams/database-relationships.md)

## Migration coverage

| Versions | Changes |
|---|---|
| V1–V3 | Persistent schema, constraints/indexes and legacy SLA target capture |
| V4–V6 | Student email delivery, four priority strategies and explicit criticality |
| V7–V9 | Automatic classification, problem templates and transfer requests |
| V10–V12 | Resolution candidate index, reported context and service description |
| V13–V16 | Dynamic service compatibility, reported service, message receipts and dynamic category/skill support |
| V17–V19 | Reported scope/template category, password recovery and internal staff channels |

Exact definitions are in `backend/src/main/resources/db/migration`. Do not edit applied migration files or run repair to bypass checksum failures.

## Submission data

`bits-itsm/database/submission-demo.sql` contains schema, constraints, indexes, sequences, settings and anonymized sample records. Restore into a new empty database as described in the root README. `submission-demo.sha256` records the export checksum; Git attributes preserve its line endings.

The export contains 23 demonstration accounts and 46 incidents. Names/emails, original password hashes and free-text narratives were replaced in an isolated clone. Chats, reset tokens and notification rows were removed. The original database was not modified. Shared demo credentials are for local evaluation only.

Dates and SLA snapshots remain historical November 2026 data. Background monitoring may emit overdue events on later startup. This export is not a production backup and cannot recover original personal data.

## Test isolation

Use the separate `bitsolve_restore_verify_chat_submission` database for the focused suite in the README. Other opt-in suites can truncate their test tables or create disposable databases. Never point tests at the live application database. Skipped tests are not passing tests.
