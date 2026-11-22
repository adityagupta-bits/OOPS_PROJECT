# React frontend

## Structure and navigation

[main.jsx](../bits-itsm/frontend/src/main.jsx) mounts [App.jsx](../bits-itsm/frontend/src/App.jsx). App owns session/account state, role-specific navigation and the portal shell. This is a React/Vite application with its own page selection, not a claimed React Router or Next.js implementation. Spring's FrontendController forwards portal URLs to index.html in the packaged application.

| Feature | Source | Backend boundary |
|---|---|---|
| Student home | [OperationsDashboard.jsx](../bits-itsm/frontend/src/OperationsDashboard.jsx) | Authorized reports, notification and option APIs |
| Reporting | [ReportIncident.jsx](../bits-itsm/frontend/src/ReportIncident.jsx) | Incident create and enabled templates/options |
| Lists and details | [LiveIncidents.jsx](../bits-itsm/frontend/src/LiveIncidents.jsx) | Filtered incident views and lifecycle actions |
| Administration | [AdminManagement.jsx](../bits-itsm/frontend/src/AdminManagement.jsx) | Users, catalog, classification and settings |
| Priority model | [PriorityConfiguration.jsx](../bits-itsm/frontend/src/PriorityConfiguration.jsx) | Saved configuration and preview |
| Assignment/team | [AssignmentManagement.jsx](../bits-itsm/frontend/src/AssignmentManagement.jsx) | Eligibility, workloads, requests and reviews |
| Availability | [TechnicianAvailability.jsx](../bits-itsm/frontend/src/TechnicianAvailability.jsx) | Persisted availability with shared updates |
| Conversations | [IncidentConversation.jsx](../bits-itsm/frontend/src/IncidentConversation.jsx) | Student and internal channels/read positions |
| Notifications | [Notifications.jsx](../bits-itsm/frontend/src/Notifications.jsx) | Recipient-specific list and read operations |
| Passwords | [PasswordSettings.jsx](../bits-itsm/frontend/src/PasswordSettings.jsx) | Change/reset and administrator code issuance |

## Student experience

The live student dashboard emphasizes open requests, resolutions ready for review, recent requests and reporting. It does not expose administrator workforce queues as student responsibilities. ReportIncident collects title, description, category and optional service/urgency/scope. Templates assist entry while preserving editable text; they do not guarantee classification independently of the backend's evidence checks.

The resulting incident view shows effective status, classification, priority and captured deadlines. Students can follow their own conversations, respond to information requests, verify or reject resolution, submit feedback and withdraw eligible NEW reports. A disabled button is only a convenience; the server rechecks each action.

## Technician experience

The dashboard presents assigned work and workload, with AVAILABLE/BUSY/UNAVAILABLE persisted by the backend. Successful preference changes update the shared visible state; failed requests display errors. Other sessions retrieve current state through their normal API refresh. Assigned reports remain assigned when availability changes.

Primary technicians use Start Work, investigation notes, information requests and resolution actions. Team requests support transfers and collaborators with reviewed decisions. Collaborators can participate in authorized work without inheriting primary-owner powers. IncidentManagementActions and shared dialogs present the appropriate actions for role and state.

## Administrator experience

All Incidents provides search, status/priority/category/service/date filters, sort selection, SLA display and detail navigation. Management actions include assignment, priority override, escalation, team requests, collaborators and audit history. Service Setup creates/edits real catalog services, validates criticality before activation and exposes backend errors. Classification Review loads eligible services from the catalog and provides a direct setup route when configuration is empty.

Priority configuration is located with Assignment Strategies. SLA policy editing controls future captured targets; it is not a UI countdown reset. Manage Users and technician profiles configure enabled status, skills and capacity with backend authorization.

## API state and failure handling

[api.js](../bits-itsm/frontend/src/api.js) sends cookies with credentials:include. apiSend obtains a CSRF token and uses the server-provided header name for JSON mutations. Requests have a 15-second timeout. A timed-out mutation warns that saving may already have completed, so users should inspect records before resubmitting. It does not automatically retry writes.

HTTP 401 broadcasts session-expired state to App, except the initial me/login checks. Expected backend messages are surfaced for validation/permission failures. Some 5xx/malformed responses are deliberately shown as a generic connection error; not every server exception is exposed verbatim. Forms use pending, success and error states rather than assuming persistence from a click.

Dashboards use Promise.allSettled so a failed section can show its own error. Empty lists are distinct from loading states. Refresh/poll cleanup prevents late updates after a component unmounts. End-to-end behavior of every rendered dialog is not established by the small Node helper test suite.

## Time, charts and updates

[SlaCountdown.jsx](../bits-itsm/frontend/src/components/SlaCountdown.jsx) and [slaCountdown.js](../bits-itsm/frontend/src/slaCountdown.js) calculate display values from server timestamps. Resolved SLA outcomes use captured completion timestamps. Display ticking changes neither priority nor database deadlines.

[IncidentReportsChart.jsx](../bits-itsm/frontend/src/IncidentReportsChart.jsx) aggregates authorized incident dates into campus-time daily totals. Historical demo records require a date range covering November 2026. Operational views combine incident, catalog, user, workload and request APIs; student views use their own scope. These are descriptive counts, not performance benchmarks.

The notification shortcut near the profile polls unread count; the operations dashboard refreshes visible data every 30 seconds. Conversation updates also use polling. “Real-time” here means periodically refreshed persisted data, not server-push WebSockets or guaranteed zero latency.

## Components and styling

Shared components under src/components/ui include Modal, Badge, PageHeader, EmptyState, StatCard and Toast. Typography/color/spacing are established in index.css and feature stylesheets, without a new UI framework. Compact tables intentionally overflow horizontally on narrow viewports rather than overlapping SLA/date columns. Modal behavior and keyboard/focus support should still be checked with a browser and assistive technology before claiming a complete accessibility audit.

## Development previews versus persisted application

demoData.js and preview components are retained because App imports them for explicitly labeled development previews. They are not dead files solely because they contain fictional records. Preview interactions must not be presented as saved backend changes. Use authenticated demo accounts from the README to demonstrate persistence; do not mix preview screenshots with live evidence.

## Build and verification

Run npm ci, npm test and npm run build in bits-itsm/frontend. The tests use Node's built-in test runner for helper logic and API behavior; they are not a full browser component suite. Vite's bundle-size warning is known and non-blocking. See [verification](VERIFICATION.md) for the exact observed result, and [API reference](API_REFERENCE.md) for contracts.
