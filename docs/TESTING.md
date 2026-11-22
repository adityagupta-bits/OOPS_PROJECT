# Testing and reproducibility

## Test boundaries

The repository contains Java domain/service/security tests, opt-in PostgreSQL persistence tests and Node-based frontend helper tests. A green default build does not mean skipped database tests passed. Historical submission results are preserved in [VERIFICATION.md](VERIFICATION.md); the current documentation pass records its own checks separately.

| Area | What is exercised | Limitation |
|---|---|---|
| Classification and priority | Keyword matching, exceptions, strategy validation/calculation | Not arbitrary natural-language comprehension |
| Assignment | Skill/availability/capacity, strategies and persisted workflows | Concurrency is not a production load benchmark |
| Lifecycle | Transitions, ownership, resolution/verification and disposition | Does not cover every browser interaction |
| Security | Role/participant checks, sessions, CSRF and password rules | Not an external penetration test |
| SLA/events | Deadline states, observer delivery and scheduled escalation behavior | SMTP delivery needs separate credentials/environment |
| Frontend | API helpers, filters, templates, service validation and countdowns | Node tests do not render React screens |
| Database | Real PostgreSQL and Flyway-backed opt-in suites | Must use disposable/isolated databases |

## Standard commands

From bits-itsm/frontend:

```powershell
npm ci
npm test
npm run build
```

From bits-itsm/backend with JAVA_HOME pointing to JDK 25:

```powershell
./mvnw.cmd verify
```

On Linux/macOS use sh ./mvnw verify. The -o option is only suitable when Maven dependencies are already cached. Build the frontend first to include current assets in the JAR. Docker's -DskipTests image build is not a substitute for verify.

## Focused PostgreSQL integration suite

Restore the anonymized export into a **new isolated** database named bitsolve_restore_verify_chat_submission, following the README's ON_ERROR_STOP restore procedure. Configure DB_USER and DB_PASSWORD privately. Do not point tests at bitsolve or another live database.

```powershell
$env:BITSOLVE_DB_TESTS='true'
$env:BITSOLVE_TEST_DB_URL='jdbc:postgresql://localhost:5432/bitsolve_restore_verify_chat_submission'
./mvnw.cmd '-Dtest=AutomaticProcessingPersistenceTests,ServiceSetupPersistenceTests,InternalStaffConversationPersistenceTests,TechnicianAvailabilitySyncPersistenceTests' test
```

The historical focused run passed 24 tests: automatic processing 16, service setup 3, internal conversations 4, availability synchronization 1. These tests verify actual persistence/authorization in the isolated clone, including custom services, automatic priority/SLA/assignment and staff privacy. Read each test's setup before running broader opt-in classes: some intentionally truncate test tables or create/drop disposable databases. A database name check is a safeguard, not permission to use production.

## Application smoke test

Build the frontend and JAR, then run on an unused port with DB_URL pointing to the restored demo. Check /api/health and /. For each role obtain /api/auth/csrf, retain cookies, POST /api/auth/login and request /api/auth/me. Use the returned CSRF header for mutations. Read-only smoke checks do not establish all lifecycle writes; use isolated tests for those.

Inspect Flyway validation/startup logs rather than repairing checksum failures. Inspect npm/Maven exit codes and reports; a log containing “tests” alone is not proof of a pass. Never record credentials, reset tokens or session cookies in evidence.

## Documentation validation

Check every relative Markdown target and Java source link, parse fenced Mermaid blocks, compare the expanded API inventory to controller mappings (including array aliases), and compare the SQL export SHA-256 with submission-demo.sha256. Review git diff --check and git diff --name-only before staging. Documentation-only changes must not alter source, migration or export hashes.

Remaining verification gaps include complete browser accessibility/responsive walkthroughs, production HTTPS/proxy operation, external SMTP and sustained load/failover testing. No performance or availability metric is claimed.
