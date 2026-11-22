# Technical documentation

Use the root [setup guide](../README.md) to restore and run BITSolve.

| Document | Contents |
|---|---|
| [Architecture](architecture.md) | Layers, modules, security and deployment |
| [Workflows](workflows.md) | Automatic processing, technician work and administration |
| [Database](database.md) | Tables, migration coverage and export semantics |
| [Object-oriented design](object-oriented-design.md) | Implemented abstractions and patterns |
| [API routes](api-routes.md) | Current controller route inventory |
| [Verification](VERIFICATION.md) | Actual checks and limitations |

## Diagrams

1. [System architecture](diagrams/architecture.md)
2. [Automatic submission sequence](diagrams/automatic-processing.md)
3. [Incident lifecycle](diagrams/incident-lifecycle.md)
4. [Database relationships](diagrams/database-relationships.md)
5. [Domain and strategy classes](diagrams/domain-classes.md)

Diagrams use Mermaid and render directly on GitHub. Their editable source is in each document. The ER diagram shows core physical relationships; logical catalog code references are explained separately. The class diagram is a selected view, not an exhaustive class list.
