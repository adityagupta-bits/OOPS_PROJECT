# Submission verification — 22 November 2026

Performed on the independent submission directory, not the live deployment.

| Check | Result |
|---|---|
| Clean dependency installation (`npm ci`) | Passed |
| Frontend unit tests | 20 passed, 0 failed |
| Frontend production build | Passed; non-blocking bundle-size warning |
| Backend `mvnw -o verify` | Passed: 146 tests passed, 71 opt-in PostgreSQL tests skipped |
| Focused isolated PostgreSQL integration suite | 24 passed, 0 failed, 0 skipped |
| Export restored into a second empty PostgreSQL database | Passed |
| Flyway V1–V19 validation against restored database | Passed on application startup and integration tests |
| Packaged application `/api/health` | HTTP 200 |
| Packaged React application `/` | HTTP 200 |
| Admin, student, technician login and subsequent `/api/auth/me` | Passed using exported demo accounts |
| Secrets/path scan of staged repository | No live credential literals or runtime files detected |

The integration suite covers automatic processing, service setup, private staff conversations and technician availability. Two fixture assumptions were corrected before the passing run: the audit assertion now follows the saved priority strategy, and the availability test isolates the tested technician from alternative eligible technicians. No application business rule was changed for packaging.

The source database contained 23 users and 46 incidents when exported. A private local backup was restored into an isolated database before anonymization. The submitted SQL includes schema, sequences, migration checksums and saved configuration. Its demo accounts all use a new demonstration-only BCrypt password; original hashes, reset tokens, conversations and recipient notifications are absent. Personal/free-text narratives were replaced. No anonymization or test writes were performed against the original database.

The packaged application was smoke-tested on port 8081 to avoid interrupting the user's existing server on 8080. The README uses port 8080 for a normal standalone setup.

Limitations: this verification did not perform a complete visual browser walkthrough of every page or enable SMTP delivery. Historical demonstration deadlines remain historical; background monitoring can produce new overdue events after startup. The 71 skipped tests are not claimed as passing; 24 selected database tests were explicitly enabled in the separate integration run.
