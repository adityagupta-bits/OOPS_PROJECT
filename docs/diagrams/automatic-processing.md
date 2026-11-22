# Automatic incident processing

```mermaid
sequenceDiagram
    actor Student
    participant UI as React
    participant API as IncidentService
    participant Classifier as Classification service
    participant DB as PostgreSQL catalog/settings
    participant Priority as Priority and SLA services
    participant Assignment as AssignmentService
    Student->>UI: Submit incident
    UI->>API: POST /api/incidents with session and CSRF
    API->>API: Validate and create NEW incident
    API->>Classifier: processSubmission
    Classifier->>DB: Read enabled catalog and configured rules
    Classifier->>Classifier: Match service, evaluate impact and urgency
    Note over Classifier: Other fallback uses configured service and defaults
    alt Ambiguous, conflicting or incomplete evidence/configuration
        Classifier->>DB: Record review reason and pending classification
    else Classification complete
        Classifier->>Priority: Calculate priority and capture SLA
        Priority->>DB: Read model and SLA policy
        Classifier->>DB: Persist classification evidence and snapshots
        API->>Assignment: assignPending
        Assignment->>DB: Check active, AVAILABLE, capacity and required skill
        alt Eligible candidate exists
            Assignment->>DB: Assign and record history/audit/notification
        else No eligible candidate
            Assignment->>DB: Retain CLASSIFIED incident pending assignment
            Note over Assignment: Scheduled worker may retry
        end
    end
    API-->>UI: Persisted incident state
    UI-->>Student: Show actual priority, status and SLA
```

No administrator approval occurs in the successful automatic path. General-support fallback uses random available allocation without a specialist skill requirement. Missing technicians do not erase classification or SLA results.
