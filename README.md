## Group 39

| Name | ID |
| --- | --- |
| Aditya Gupta | 2025A7PS0923H |
| Lakshya Goyal | 2025A7PS0120H |
| Aditya Dash | 2025ADPS0218H |
| Kush Gupta | 2025A7PS2420H |

# BITSolve — Campus IT Service Management

Java 25 / Spring Boot 4.1.2 / React 19 / Vite / PostgreSQL 18.

BITSolve supports Student, Technician and Administrator workflows: incident reporting, catalog-driven classification, configurable priority calculation, skill/capacity-based assignment, SLA countdowns and background monitoring, escalation, transfer requests, collaboration, student conversations, internal staff chat, resolution verification, feedback, notifications and audit history.

## Purpose and objectives

Campus support reports need a traceable route from a student’s description to a responsible technician, a response target and a verified outcome. BITSolve combines that workflow with configurable automation and exception review. It aims to reduce routine manual allocation while retaining accountable administrator overrides and student visibility.

| Role | Main capabilities |
|---|---|
| Student | Report and track incidents, converse with support, verify/reject a resolution, leave feedback, withdraw eligible new reports |
| Technician | Manage availability, start assigned work, record investigation, request information/transfer/collaboration, resolve incidents |
| Administrator | Manage catalog/users/policies, inspect exceptions, assign/reassign, approve team requests, override/escalate with history |

## Submission contents

- `bits-itsm/backend`: complete Java source, Maven wrapper, tests and Flyway V1–V19.
- `bits-itsm/frontend`: React source, frontend tests and locked npm dependencies.
- `bits-itsm/database/submission-demo.sql`: PostgreSQL schema and anonymized demonstration data, including saved service, priority, assignment and SLA settings.
- [Architecture](docs/architecture.md), [workflows](docs/WORKFLOWS.md), [OOP design](docs/OOP_DESIGN.md), [API reference](docs/API_REFERENCE.md) and [academic report](docs/PROJECT_REPORT.md).
- `docs/VERIFICATION.md`: checks performed for this submission.

This snapshot includes the latest local source changes without importing the old repository history. Local logs, credentials, personal conversations and password-reset tokens are excluded.

## Run locally

Prerequisites: JDK 25 (`JAVA_HOME` pointing to the JDK), Node.js 24/npm, PostgreSQL 18 with `psql` and `createdb` on PATH. Maven is supplied through the wrapper; initial builds need internet access to download dependencies.

### 1. Restore the demonstration database

Run from the repository root. **Use a new empty database, never your existing BITSolve database.** `createdb` will fail if this database already exists; do not continue after that failure.

```powershell
createdb -h localhost -U postgres bitsolve_submission
psql -X -h localhost -U postgres -d bitsolve_submission -v ON_ERROR_STOP=1 -f bits-itsm/database/submission-demo.sql
```

Supply your own PostgreSQL password when prompted, or configure your PostgreSQL password file. No database server password is included.

### 2. Build React, then the backend

```powershell
cd bits-itsm/frontend
npm ci
npm test
npm run build
cd ../backend
./mvnw.cmd verify
```

On Linux/macOS use `sh ./mvnw verify`. React's production assets are included in the executable JAR when built in this order. Build output is deliberately not committed.

### 3. Start the application

From `bits-itsm/backend`, in PowerShell:

```powershell
$env:DB_URL='jdbc:postgresql://localhost:5432/bitsolve_submission'
$env:DB_USER='postgres'
$secret=Read-Host 'Your PostgreSQL password' -AsSecureString
$env:DB_PASSWORD=[System.Net.NetworkCredential]::new('', $secret).Password
java -jar target/bitsolve-backend-0.0.1-SNAPSHOT.jar
```

Open **http://localhost:8080**. Health: **http://localhost:8080/api/health**.
Flyway validates the restored migration history; Hibernate validates the schema. Do not run Flyway repair or edit applied migrations.

For frontend development, run `npm run dev` in `bits-itsm/frontend` while the backend runs on 8080. Vite proxies `/api` to the backend.

### Demonstration login accounts

All accounts in this **anonymized export only** use password **`SubmissionDemo2026!`**:

| Role | Email |
|---|---|
| Administrator | `demo.admin1@hyderabad.bits-pilani.ac.in` |
| Student | `demo.student2@hyderabad.bits-pilani.ac.in` |
| Technician | `demo.technician6@hyderabad.bits-pilani.ac.in` |

Select the corresponding role on the login form. Other demo usernames follow `demo.<role><id>@hyderabad.bits-pilani.ac.in`; inspect Manage Users for the full list. Passwords in the original database were not changed. These shared demonstration credentials must never be used for a public deployment.

## Database and demonstration notes

The export contains 23 anonymized accounts and 46 incidents at packaging time. Dates, service mappings, status, priority and SLA snapshots are retained; names, emails, incident narratives and free-text history are replaced. Chats, notifications and password-reset tokens are removed. Feedback is explicitly labeled demonstration data. The live database was only read, not modified.

Historical incident dates remain in November 2026. As real time advances, open deadlines may become overdue and the normal background SLA worker can create notifications. Graph filters should be adjusted to include these historical dates. This is expected, not a timer reset.

Automatic processing depends on enabled services with configured criticality, matching evidence, saved rules and active AVAILABLE technicians with suitable skills and capacity. Ambiguous reports or missing configuration enter exception queues. Busy/Unavailable technicians do not receive routine automatic assignments. Inspect configuration under Administrator → IT Services, Urgency & Impact Rules, Assignment Strategies (including priority configuration), and SLA Policies.

For a fresh installation, Flyway can initialize an empty schema, but an initial administrator and valid service/rule settings must be provisioned before normal operation. Public registration creates students only. Use the supplied restore for the reproducible evaluation setup.

## Tests

`npm test` runs frontend unit tests. `./mvnw.cmd verify` runs backend tests; PostgreSQL tests are opt-in and skipped unless enabled. Never point integration tests at a live database.

For the focused transactional PostgreSQL suite, restore the export into another new database named `bitsolve_restore_verify_chat_submission`, configure `DB_USER` and `DB_PASSWORD`, then run:

```powershell
$env:BITSOLVE_DB_TESTS='true'
$env:BITSOLVE_TEST_DB_URL='jdbc:postgresql://localhost:5432/bitsolve_restore_verify_chat_submission'
./mvnw.cmd '-Dtest=AutomaticProcessingPersistenceTests,ServiceSetupPersistenceTests,InternalStaffConversationPersistenceTests,TechnicianAvailabilitySyncPersistenceTests' test
```

Other PostgreSQL test classes may deliberately truncate test tables or create/drop disposable databases; read their setup before enabling them. Separate test databases are required.

## Optional email

In-app notifications work without SMTP. Actual email delivery additionally requires `MAIL_USERNAME`, `MAIL_APP_PASSWORD` and `APP_BASE_URL`. Keep these private; no SMTP credentials are supplied. Do not configure a real mailbox for demonstration data.

Development-only preview URLs show fictional read-only UI previews; use real login accounts above to demonstrate persistence and backend operations.


## Configuration and Docker

| Variable | Purpose |
|---|---|
| DB_URL | JDBC PostgreSQL connection; otherwise DB_HOST/DB_PORT/DB_NAME compose the URL |
| DB_USER / DB_PASSWORD | Private database credentials; DB_USERNAME is a fallback alias |
| PORT | Backend port, default 8080 |
| SESSION_COOKIE_SECURE | Enable for HTTPS; prod defaults true |
| SPRING_PROFILES_ACTIVE | prod for deployment settings, including disabled demo seeding |
| MAIL_USERNAME / MAIL_APP_PASSWORD / APP_BASE_URL | Optional SMTP sender and public application URL |

The source manifest uses Java 25, Spring Boot 4.1.2, React ^19.3.1 and Vite ^8.3.5; package-lock.json fixes npm dependency resolutions. PostgreSQL 18 and Node 24 were used for the submission checks. JPA/Hibernate and Spring Security versions are managed by Spring Boot rather than separately specified here.

Build from bits-itsm:

```powershell
docker build -t bitsolve .
docker run --rm -p 8080:8080 --env-file .env bitsolve
```

Create the private .env with DB_URL, DB_USER and DB_PASSWORD; never commit it. PostgreSQL must be reachable from the container: localhost means the container itself, not the host. Docker Desktop can use host.docker.internal for a host database. The image does not include or restore PostgreSQL. It runs as UID 10001 and skips tests during image build. For a local HTTP-only demonstration set SESSION_COOKIE_SECURE=false explicitly; deploy real installations behind HTTPS with secure cookies enabled. Docker runtime/HTTPS were not reverified in this documentation pass.

Email domain validation is not mailbox ownership verification. Shared demo credentials and historical anonymized records are for local evaluation only. SMTP delivery needs separate verification and must not target real recipients from demo data.

## Documentation

- [Architecture](docs/architecture.md): layers, security boundaries, events and deployment.
- [Java OOP](docs/OOP_DESIGN.md): actual classes, snippets and design patterns.
- [Workflows](docs/WORKFLOWS.md): automation, exceptions, lifecycle and team operations.
- [Database](docs/DATABASE.md): tables, relationships, migrations and restore constraints.
- [API reference](docs/API_REFERENCE.md): expanded routes, access and DTOs.
- [Frontend](docs/FRONTEND.md): portals, API state, components and preview boundaries.
- [Testing](docs/TESTING.md): reproducible commands and isolation.
- [Verification](docs/VERIFICATION.md): observed results, cleanup decisions and limits.
- [Academic report](docs/PROJECT_REPORT.md): concise report prepared for a 20–25-page layout.
