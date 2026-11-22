# Incident lifecycle

```mermaid
stateDiagram-v2
    [*] --> NEW: Submit
    NEW --> NEW: Review required
    NEW --> CLASSIFIED: Automatic classification or authorized correction
    CLASSIFIED --> ASSIGNED: Eligible technician assigned
    ASSIGNED --> ACKNOWLEDGED: Technician acknowledges
    ACKNOWLEDGED --> IN_PROGRESS: Start investigation
    IN_PROGRESS --> RESOLVED: Record resolution
    RESOLVED --> VERIFIED: Reporter confirms
    RESOLVED --> IN_PROGRESS: Reporter rejects resolution
    VERIFIED --> CLOSED: Close verified incident
    CLOSED --> [*]
```

This shows the normal lifecycle, not every authorized exceptional transition. `Needs Review` and pending assignment are flags/conditions, not additional enum states. Reassignment/collaboration and information requests have their own rules and do not introduce a `WAITING` enum status. Withdrawal or invalid/duplicate disposition can close eligible non-resolved reports without a successful resolution; history is retained.
