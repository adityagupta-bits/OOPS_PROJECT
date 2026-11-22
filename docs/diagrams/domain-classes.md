# Domain and strategy classes

```mermaid
classDiagram
    User <|-- Student
    User <|-- Technician
    User <|-- Administrator
    Incident --> User : reporter
    Incident --> Technician : primary technician
    Incident "1" *-- "0..*" IncidentHistory
    Incident "1" *-- "0..*" IncidentCollaborator
    IncidentCollaborator --> Technician
    IncidentClassificationService --> CatalogService
    IncidentClassificationService --> PriorityService
    PriorityService --> PrioritySettings
    PrioritySettings --> PriorityStrategy : selects
    PriorityStrategy <|.. WeightedPriorityStrategy
    PriorityStrategy <|.. MultiplicativePriorityStrategy
    PriorityStrategy <|.. MatrixPriorityStrategy
    PriorityStrategy <|.. RuleBasedPriorityStrategy
    AssignmentService --> WorkloadService
    AssignmentService --> AssignmentStrategy
    AssignmentStrategy <|.. SkillBasedAssignment
    AssignmentStrategy <|.. LoadBasedAssignment
    AssignmentStrategy <|.. RandomAvailableAssignment
    EventBus --> IncidentObserver : dispatches
    class PriorityStrategy {
        <<interface>>
        calculate(impact, urgency, criticality)
    }
    class AssignmentStrategy {
        <<interface>>
        assign(incident, candidates)
    }
    class IncidentObserver {
        <<interface>>
        onEvent(event)
    }
```

Selected design relationships. Service dependencies are not database tables. Dynamic catalog code value objects and transport DTOs are omitted for readability; see [OOP design](../object-oriented-design.md).
