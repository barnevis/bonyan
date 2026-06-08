# Core

## Introduction

The core is the smallest and most stable part of the architecture. The core's responsibility is **coordination** only — not implementing logic.

The core is the only part whose lifecycle matches the lifecycle of the entire system. The core always runs first and stops last.

The core does not know:
* What the product does
* What the UI looks like
* What environment it runs in
* What logic modules and plugins contain

## Core Responsibilities

The core has nine responsibilities and must not exceed them:

1. System lifecycle management
2. Loading and managing the lifecycle of parts
3. Manifest and contract validation
4. Service registry
5. Event bus management
6. Error management
7. Shared state maintenance
8. Configuration management
9. Internal logging

## System Lifecycle

The core manages the startup and shutdown sequence of the entire system.

### Startup

1. Initialize the core and internal logger
2. Read `bootstrap.json`
3. Check architecture version
4. Load core-required plugins
5. If a logger plugin exists, forward logs from the internal logger to it
6. Identify the runtime environment
7. Validate and load the appropriate platform
8. Validate and load plugins in order
9. Validate and load modules
10. Validate and load the UI
11. System is ready

### Shutdown

The shutdown sequence is the reverse of startup so no part is stopped without being ready:

1. Stop the UI
2. Stop modules
3. Stop plugins
4. Stop core plugins
5. Stop the core

## Loading and Managing Part Lifecycles

The core is responsible for loading, starting, and stopping each part independently. This responsibility is separate from system lifecycle management because each part has its own independent lifecycle.

For each part, the core goes through these steps:

1. Load ← read manifest and contract from the filesystem
2. Validate ← check structure and compatibility
3. Start ← inject configuration and register in the registry; after successful registration, publish `core:part-loaded` on the system bus
4. Stop ← notify, wait for operations to complete, remove from registry; after removal, publish `core:part-unloaded` on the system bus

If loading a part fails, the core ignores that part, logs the error, and continues loading the remaining parts.

The core checks for circular dependencies between parts before startup begins. If a circular dependency is detected, the core halts startup and throws a descriptive error.

## Internal Logger

The core has a simple internal logger that is available from the moment it runs. This logger has no external dependencies and only prints to the console or standard output.

The internal logger is used in two cases:

* Before the logger plugin is loaded — all core logs are recorded through the internal logger
* When the logger plugin is unavailable — the core falls back to the internal logger

When the logger plugin is loaded, the core automatically forwards all logs to it.

## Manifest and Contract Validation

The core validates the manifest and contract of every part before loading it. This validation happens in two phases:

**Phase One — Structure Validation** (before loading):
```text
Do manifest.json and contract.json exist?
Are all required manifest fields present?
Do versions have the correct format?
Is the part type valid?
Is the architecture version compatible?
Does the contract declared in implements exist?
Are required dependencies available and do their versions match the range declared in the manifest?
Do required config keys have values?
Are all required contract fields present?
```

**Phase Two — Implementation Validation** (after loading):
```text
Are all contract methods implemented?
Does each method's signature match the contract?
```

If phase one validation fails, the part is not loaded. If phase two validation fails, the part is removed from the registry. In both cases the core logs and announces a clear error.

## Service Registry

The core maintains a central registry where plugins, modules, the platform, and the UI are registered. This registry is the only authoritative source for accessing services.

```text
Register ← a part is registered after successful validation
Get      ← a part requests a service it needs from the registry
Remove   ← a part is removed from the registry
```

The registry is only for managing part lifecycles. Communication between parts happens only through the event bus. The core controls each part's access to the registry based on the dependencies declared in its manifest.

## Event Buses

The core manages event buses. Each part can have its own separate bus to maintain isolation and security.

### Bus Types

```text
System bus  ← public events that all parts can listen to
Private bus ← private events of each part, declared in the manifest
```

### Bus Operations

```text
Publish     ← a part announces that something happened
Subscribe   ← a part declares that it cares about an event
Unsubscribe ← a part declares that it no longer cares about an event
```

### Bus Rules

* A message can carry a `data` field
* Small data can be transferred directly in the message
* Large data must be stored in a temporary storage plugin and only its identifier passed in the message
* The publisher does not know who receives the event
* The receiver does not know who published the event
* No part can directly access another part's bus
* The core decides which events can travel from a private bus to the system bus
* Only the core can create a new bus
* The core only publishes events that are declared in the publishing part's manifest — any event outside this list is rejected

### Private Bus Creation Rules

A private bus is only created when:

* A part exchanges sensitive data that must not be published on the system bus
* A part has high-frequency internal events that are irrelevant to other parts

A part declares its need for a private bus in its manifest and the core creates it and makes it available to the part.

## Error Management

The core is responsible for containing errors that escape the boundary of any part.

```text
1. The error is contained at the same boundary
2. The core logs the error
3. The core notifies via the system bus
4. The system continues operating with reduced capability
```

The core never stops the entire system because of one part's error.

The core's error handling functions are private and are not exposed to external parts through the public API. Each part is responsible for managing its own internal errors — the core only contains errors that escape a part's boundary.

## Shared State

The core holds data that all parts need and that has no specific owner.

Shared state in the core is only for data that is truly global and not tied to any business logic. Any data related to product logic must be managed in a plugin or module.

Examples:

```text
UI language
Network connection status
```

### Shared State Rules

* Only the core can change shared state
* Shared state is not exposed to external parts through the core's public API — changing shared state happens only from inside the core, in response to specific events
* Parts can read shared state
* Every change to shared state is announced via the system bus
* Shared state must be kept to the minimum necessary
* No business data must be placed in the core's shared state

## Configuration

The core is the access point for general system settings. Settings are read from `bootstrap.json` and made available to all parts.

Important distinction:
* **Product configuration** — settings like API addresses and public keys that do not change after startup
* **User settings** — like language and theme, managed in the core's shared state or a separate plugin

### Configuration Rules

* No module or plugin may hardcode settings
* All settings must be read from the core
* Product configuration does not change after system startup
* The core must not access environment variables directly — the environment source must be injected into the core from outside. This principle keeps the core decoupled from the runtime environment and ensures its testability

## What the Core Is Not

To prevent the core from growing too large, these responsibilities are explicitly outside the core's scope:
- Business logic ← responsibility of modules
- Infrastructure ← responsibility of plugins
- Data display ← responsibility of the UI
- Runtime environment access ← responsibility of the platform
- Business data ← responsibility of plugins and modules
