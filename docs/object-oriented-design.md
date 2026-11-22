# Object-oriented design

These structures exist in the Java implementation; they are not proposed features.

| Principle / pattern | Classes | Purpose |
|---|---|---|
| Encapsulation | `Incident`, `Technician`, `PrioritySettings` | Validated transitions, availability/capacity and settings |
| Inheritance | `User`, `Student`, `Technician`, `Administrator` | Shared identity and role-specific data/behavior |
| Strategy / polymorphism | `PriorityStrategy` and four implementations | Interchangeable configured priority calculations |
| Strategy | `AssignmentStrategy`, `SkillBasedAssignment`, `LoadBasedAssignment`, `RandomAvailableAssignment` | Allocation over eligible candidates |
| Observer | `EventBus`, `IncidentObserver`, `IncidentEvent` | In-process notification/audit event handling |
| Factory | `NotificationFactory` | Event-derived notification construction |
| Repository | `IncidentRepository`, `UserRepository`, settings repositories | Persistence boundary |
| Value objects | `ServiceCode`, `Incident.ItService`, `Incident.Category`, `TechnicianSkill` | Validated dynamic identifiers |
| DTOs / records | `IncidentData`, `AssignmentData`, request/response records | Explicit API transport and validation |
| Dependency injection | Constructor-injected services | Explicit dependencies and focused testing |

Lifecycle status, priority and availability are fixed enums. Catalog service/category identifiers are string-backed; their `valueOf` helper does not make them fixed enums.

[Selected class diagram](diagrams/domain-classes.md). Framework internals and utility classes are intentionally omitted.
