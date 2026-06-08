# Architecture

## Introduction

This architecture is a modular, cross-platform structure for building software products based on JavaScript, HTML, and CSS — designed without dependency on heavy frameworks.

The primary goal is to reduce complexity, increase maintainability, improve code reuse, and simplify development by both humans and AI models.

This architecture aims to reduce dependency on frameworks, hidden behaviors, and complex abstractions — relying instead on explicit contracts, clear boundaries, and predictable structures.

## Architecture Goals

The architecture must:

* Accelerate product development
* Increase reuse of capabilities
* Reduce coupling between product parts
* Make development by humans and AI models simpler and more reliable
* Evolve over the long term without complete rewrites

## Overall Structure

The system consists of five main parts, each with a specific responsibility:

```text
        Modules
       ↙         ↘
   UI          Plugins
       ↘         ↙
          Core
            ↓
         Platform
```

* **Modules** implement the core logic and capabilities of the product
* **UI** displays data, receives user interactions, and collects user-entered data
* **Plugins** provide shared infrastructure
* **Core** coordinates between parts through the event bus
* **Platform** provides the connection to the runtime environment

## Architecture Layers

### Core

The core is the central part of the architecture, responsible for coordination and management of architecture components.

The core does not know the project's logic, is not dependent on the UI, and is not aware of runtime environment details. The core does not interfere in data transfer between parts.

Core responsibilities:

* System lifecycle management
* Loading and managing the lifecycle of parts
* Manifest and contract validation
* Service registry
* Event bus management
* Error management
* Shared state maintenance
* Configuration management
* Internal logging

### Event Bus

The event bus is the only permitted channel for communication between parts and is managed by the core.

```text
Event Bus  →  The only communication channel between all parts
```

A message can carry a `data` field. Small data can be transferred directly in the message. Large data must be stored in a temporary storage plugin and only its identifier passed in the message. Sensitive data such as tokens or passwords must not be placed in messages.

Any message whose response must be matched to the original request must carry a unique identifier. The response must return the same identifier so the sender can match the response to its request.

Any part that publishes a message and expects a response is responsible for handling the no-response case. If no response arrives within the expected time window, the part must report the error to the user.

```text
task:created      →  { data: { id: "123" } }
file:uploaded     →  { data: { id: "abc123", size: 10485760, mimeType: "image/png" } }
transactions:ready → { data: { ref: "store-key-xyz" } }
core:ready        →  no data
report:requested  →  { data: { requestId: "req-123", from: "2024-01-01", to: "2024-12-31" } }
report:ready      →  { data: { requestId: "req-123", result: {...} } }
```

### Plugins

Plugins provide shared infrastructure and capabilities across projects. Plugins are part of the system from day one and serve as the foundation for modules to operate.

Plugins are responsible for providing infrastructure, not implementing product logic.

Examples:

* Storage
* Authentication
* Routing
* AI integration
* Analytics

### Modules

Modules implement the features and core logic of the product. Each module represents an independent business capability or feature.

Examples:

* Task management
* Chat
* Projects
* Notifications

Modules can use plugins but must not depend directly on each other. Each module communicates through the event bus using message types declared in its contract.

### UI

The UI layer is responsible for displaying data, receiving user interactions, and collecting user-entered data.

The UI must not contain core logic. Business logic must live outside the UI to be testable, reusable, and executable in different environments.

All UI communication happens through the event bus:

```text
Module publishes message  →  UI is notified  →  requests data via bus  →  display is updated
```

### Platform

The platform layer is responsible for connecting the product to the runtime environment and enabling the product to run in different environments.

Examples:

* Browser
* Desktop
* Mobile

All access to runtime environment capabilities must happen through explicit interfaces and contracts.

## Foundational Principles

### 1. Simplicity Over Complexity

Simplicity is not a decorative choice — it is an architectural principle. Every part of the product must be understandable, have predictable behavior, and be developable without hidden knowledge.

Complexity is only acceptable when it creates real, long-term benefit.

### 2. Business Logic Must Be Independent

Product logic must not depend on the UI, database type, runtime environment, or external services — and must be able to run in different environments without modification.

### 3. Everything Must Communicate Through Contracts

Product components must not depend directly on each other's implementations. Communication between parts must only happen through explicit, stable contracts.

Contracts are the foundation of extensibility, replaceability, testability, and AI-driven development.

### 4. The Core Must Stay Small and Unaware

The core must not know the project's logic, UI, or runtime environment details. The smaller and more general the core, the more stable and reusable the product will be.

### 5. All Capabilities Must Be Replaceable

No part of the product should depend on a specific implementation. The storage system, authentication, UI engine, or API communication method must be replaceable without changing the core logic.

### 6. Modules Must Be Independent and Isolated

Each module must have a clear responsibility, define explicit boundaries, and be developed without direct dependency on other modules.

### 7. UI Is Only the Display Layer

The UI's job is to display data, receive user interaction, collect user-entered data, and send messages. All decisions and core rules must live outside the UI.

### 8. Architecture Must Be Understandable by Humans and AI

This architecture must also be analyzable and developable by AI models. For this reason, the product must have clear structures, small files, limited dependencies, transparent behaviors, and explicit contracts.

### 9. Error Isolation

A failure or fault in one module or plugin must not bring down the entire system. Errors must be contained at the boundary where they occur and the system must be able to continue operating, even with reduced capability.

If a part encounters a critical error during runtime and is removed from the registry, dependent parts continue operating. These parts must listen to the `core:part-failed` event and disable the capabilities that depend on the lost part.

### 10. Explicit Data Ownership

Every piece of data in the system must have one clear, single owner. No part may directly modify another part's data. Requests to change data must go through contracts and messages.

### 11. Data Transfer Must Be Explicit and Direct

Data can be transferred through the event bus as long as it is not large. If data is large, it must be stored in a temporary storage plugin and only its identifier passed in the message. Modules do not read each other's data directly. Determining what counts as large data is left to the developer; the general principle is that if transferring data through a message causes a performance problem, it should switch to the storage-and-identifier approach.

## Communication Rules Between Parts

The direction of dependencies in the product must be explicit and controlled:

```text
Modules ← Plugins ← Core ← Platform
```

* The core must not depend on plugins, modules, or any specific project
* All parts communicate only through the event bus
* Parts know each other only through contracts
* No part may depend directly on another part's internal implementation

## Final Goal

The goal of this architecture is to build a product that is fast to develop, simple to maintain, has minimal dependencies, runs in different environments, and can evolve over the long term without complete rewrites.

This architecture seeks to create a balance between simplicity, flexibility, extensibility, and long-term stability.

## Testability Principles

Testability in this architecture is a principle, not a choice. The architecture must be designed so that every part can be tested independently from the rest of the system.

### Core Principles

* Every part must be testable without the core
* Every part's dependencies must be replaceable with fakes
* Contracts are the foundation of testability — if a contract is clear, writing tests is straightforward
* The internal logic of every part must be testable independently of the runtime environment

### Contract Testing

Every implementation must be tested against its own contract. This means:

* All methods declared in the contract must have tests
* Tests must verify declared behavior, not implementation details
* If a contract changes, tests must be updated

## External Library Dependencies

Dependency on external libraries must be controlled and limited. Every external library is a dependency that makes maintenance, updates, and replacement more complex.

Dependency rules for each part:

**Core** — Must have no external libraries. The core must be completely independent to run in any environment without extra dependencies.

**Plugins** — May have external libraries because they are infrastructure and responsible for implementing specialized capabilities. External dependencies of a plugin must be declared in the manifest.

**Modules** — Must not depend directly on external libraries. If a module needs a capability from an external library, that capability must be provided through a plugin.

**Platform** — May have external libraries because it depends on runtime environment capabilities.

**UI** — May have display libraries, but must not use libraries that contain business logic.

These rules follow three architectural principles: independence of parts, replaceability, and keeping the core small.
