# Submission verification — 22 November 2026

## Documentation refresh

Architecture, workflow, database and OOP documentation were checked against the current Java implementation. The API inventory contains 97 controller mappings. All five Mermaid diagrams parsed successfully and all 122 local Markdown links resolved. Frontend tests were rerun (20 passed), and the production build passed. Source edits in this refresh were verified to change comments/whitespace only; no business logic, CSS declarations, migration or database data changed. Empty legacy SQL placeholders were removed to avoid confusing them with the actual restore export.

## Application packaging checks

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

## Final documentation overhaul � 22 November 2026

This pass changed documentation only. The Java, React, migrations, deployment files and SQL export remain unchanged. Source comparison against the original workspace found no differences in the corresponding application source/configuration files. No original database was connected to or modified during this pass.

### Checks executed in this pass

| Check | Observed result |
|---|---|
| `npm test` in `bits-itsm/frontend` | 20 passed, zero failed |
| `npm run build` | Passed; 510.96 kB JavaScript chunk triggers the existing >500 kB warning |
| `./mvnw.cmd -o verify` with JDK 25 | BUILD SUCCESS; 217 discovered, 146 passed, 71 opt-in tests skipped, zero failures/errors |
| Relative Markdown/source links | 142 resolved |
| Mermaid syntax | All 7 fenced diagrams parsed |
| SQL export SHA-256 | Matches `submission-demo.sha256` |
| Source comparison with original workspace | No differing application source/configuration files |
| Diff scope | README and docs only; no source/migration/export changes |

The earlier 24-test PostgreSQL integration result remains historical evidence; it was not rerun during this documentation-only pass. Docker runtime, SMTP, full browser/accessibility coverage and PDF pagination were not tested here. PROJECT_REPORT.md contains 25 planned page sections with explicit breaks; final pagination still requires checking after conversion. The existing separate 48-page local PDF was preserved.

### Cleanup decisions

| Previous file | Decision | Destination / reason |
|---|---|---|
| `docs/README.md` | MERGE / REMOVE duplicate | Root README now provides the complete documentation index |
| `docs/api-routes.md` | MERGE / REPLACE | API_REFERENCE.md expands 100 controller routes, aliases, DTOs, access and failure conditions; logout is documented separately |
| `docs/object-oriented-design.md` | MERGE / REPLACE | OOP_DESIGN.md retains the real patterns and adds source snippets, definitions and limits |
| `docs/database.md` | MERGE / RENAME | DATABASE.md adds migration links, keys, consistency and restore details |
| `docs/workflows.md` | MERGE / RENAME | WORKFLOWS.md adds exception handling, team/lifecycle details and correct saved SLA targets |
| `docs/diagrams/architecture.md` | MERGE / REMOVE duplicate | Diagram retained inline in architecture.md |
| `docs/diagrams/automatic-processing.md` | MERGE / REMOVE duplicate | Sequence retained inline in architecture.md |
| `docs/diagrams/database-relationships.md` | MERGE / REMOVE duplicate | ER diagram retained inline in DATABASE.md |
| `docs/diagrams/domain-classes.md` | MERGE / REMOVE duplicate | UML retained inline in OOP_DESIGN.md |
| `docs/diagrams/incident-lifecycle.md` | MERGE / REMOVE duplicate | State diagram retained inline in WORKFLOWS.md |

New distinct guides are FRONTEND.md (live components/API state/preview limits), TESTING.md (commands and isolation) and PROJECT_REPORT.md (concise academic report). README and architecture.md were expanded; this verification record preserves the earlier results rather than replacing them with newly invented counts.

KEEP: all source, tests, Maven wrapper, lockfiles, Docker configuration, Git attributes, Flyway migrations and the anonymized SQL/checksum. Imported development preview modules are retained because application code uses them. No active AGENTS/CLAUDE/GEMINI instructions or tracked prompt/handover dumps were found in this submission repository. There were no untracked submission files at the start of this pass. Ignored build outputs remain local and untracked; existing .gitignore already covers logs, credentials, dependencies and build directories, so no extra blanket exclusions were needed.

REVIEW / retained outside submission: original-workspace historical notes, logs, temporary authoring files and the existing project-report PDF were not deleted or added to the repository. Their local history may still be useful, and it is outside this cleanup's submission scope. The previously removed presentation PDF was not reintroduced.

### Final documentation structure

```
README.md
bits-itsm/
  backend/
  frontend/
  database/
  Dockerfile
docs/
  architecture.md
  OOP_DESIGN.md
  WORKFLOWS.md
  DATABASE.md
  API_REFERENCE.md
  FRONTEND.md
  TESTING.md
  VERIFICATION.md
  PROJECT_REPORT.md
```

### Quality assessment

These are editorial assessments, not measured software-quality metrics: completeness **9/10** (all requested subjects and expanded endpoints covered); accuracy **9/10** (source/DTO links and corrected model/SLA/security claims, with runtime limits explicit); navigability **9/10** (one root index, consolidated diagrams and verified links); academic usefulness **9/10** (real OOP examples and a bounded report outline). Remaining work for a polished printed submission is render-and-review of the concise report after PDF conversion; no new PDF is claimed by this Markdown documentation task.
