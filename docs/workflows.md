# Application workflows

## Submission and automatic processing

1. A student submits a title, description, category and optional context/template. Suggested urgency is context, not a direct priority override.
2. The backend validates and creates the incident, then matches enabled services with valid criticality against configured keywords in the title/description.
3. Matching normalizes case, Unicode hyphens and Wi-Fi variants. Distinct matching keywords determine score; tied services remain ambiguous. For the combined LAN/Wi-Fi template, unambiguous description evidence can identify the actual network service.
4. Impact and urgency come from configured evidence rules. An unmatched, unambiguous report explicitly categorized Other can use the enabled `OTHER` service and administrator-configured default impact/urgency. Valid fallback criticality is required.
5. Complete, consistent classification enters the saved priority strategy and captures SLA targets/deadlines. Evidence is recorded.
6. Automatic allocation runs immediately. Known services require an active AVAILABLE, skill-qualified technician with spare capacity. General-support fallback selects randomly from active AVAILABLE technicians with capacity without requiring a specialist skill.

Routine incidents require no administrator approval. Notifications do not pause processing. Unknown/tied services, conflicting hints or scope, missing evidence/configuration and disabled automation can require Classification Review. A classified incident with no eligible technician retains priority/SLA while awaiting assignment; the worker can retry it.

[Submission sequence](diagrams/automatic-processing.md)

## Priority and SLA

Priority configuration currently appears under **Administrator → Assignment Strategies**. The saved strategy can be Weighted, Multiplicative, Matrix or Rule-based. It is not chosen by the student.

Weighted score is `50 × (impactWeight × impact + urgencyWeight × urgency + criticalityWeight × criticality)`, using LOW/MEDIUM/HIGH ordinals 0/1/2 and configured thresholds. Weights are non-negative and sum to 1. Other models use their own configured thresholds, matrix or ordered rules.

The submission's saved response/resolution policies in minutes are P1 **5/30**, P2 **15/120**, P3 **60/480**, P4 **240/1440**. These are editable settings, not universal constants. Existing deadlines are captured snapshots; priority override preserves them.

The frontend displays remaining time from captured timestamps. Server monitoring runs independently of login, normally every five seconds. It records warnings at 80% elapsed and breaches after deadlines. Unacknowledged P1 incidents escalate after the response deadline through Team Lead, System Administrator and IT Manager, with a default five-minute step interval. Administrator manual escalation is separate and audited. Countdown is an SLA target, not a predicted repair duration.

## Technician and administrator actions

The primary technician acknowledges an assigned incident, starts investigation, records notes, requests information and supplies a resolution. The reporter verifies or rejects the resolution. Rejection returns to investigation; verification precedes normal closure.

AVAILABLE permits new automatic allocations; BUSY and UNAVAILABLE exclude them. Existing assignments remain attached when availability changes. Profile and administrator workload views use backend state.

- **Transfer:** a reviewed request changes the primary technician when approved.
- **Collaboration:** approval adds an assisting technician/responsibility while preserving the primary owner.
- **Administrator:** can inspect, assign/reassign, manage collaborators, override priority, escalate and inspect history. Operations enforce eligibility and required reasons.
- **Classification Review:** corrects actual exceptions with an audit reason and continues the existing processing pipeline; it is not routine approval.

## Communication and notifications

Student Conversation is incident-scoped and accessible to the reporter and authorized support participants. Internal Staff Chat is limited to administrators, primary technician and authorized collaborators. Urgent staff instructions are administrator-only. Read positions are persisted per channel. There is no global all-staff broadcast or claimed WebSocket transport.

Notifications persist per recipient with read state, filtering and incident navigation. In-app delivery does not need SMTP. Student email delivery requires a configured sender and uses the persisted outbox.

## Catalog, closure and feedback

Administrators create services with a unique code, name, description, keywords and criticality. Activation requires criticality; enabled configured services appear in Classification Review and matching. Disabled services are excluded.

Withdrawal and invalid/duplicate closure retain records without claiming successful resolution. Normal resolution, verification and closure are distinct transitions. Feedback belongs to the reporting student and incident. Sample feedback in the export is labeled demonstration data.

[Lifecycle diagram](diagrams/incident-lifecycle.md)
