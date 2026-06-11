# Core

## Introduction

The Core is the smallest and most stable part of the architecture. The Core's responsibility is solely **coordination**, not logic implementation.

The Core is the only part whose lifecycle is identical to the lifecycle of the entire system. The Core is always executed first and stopped last.

The Core does not know:
* What the product does
* What the user interface looks like
* In what environment it runs
* What logic the modules and plugins contain

## Core Responsibilities

The Core has nine responsibilities and should have no more:

1. System lifecycle management
2. Loading and managing the lifecycle of parts
3. Manifest and contract validation
4. Service registry
5. Channel management
6. Error management
7. Maintaining shared state
8. Configuration management
9. Internal logging

## System Lifecycle

The Core manages the startup and shutdown sequence of the entire system.

### Startup

1. Initializing the Core and internal logging
2. Reading `bootstrap.json`
3. Checking the architecture version
4. Loading plugins required by the Core
5. Transferring logs from internal logging to a log plugin (if present)
6. Identifying the execution environment
7. Validating and loading the appropriate platform
8. Validating and loading plugins in order
9. Validating and loading modules
10. Validating and loading the user interface
11. The system is ready

### Shutdown

The shutdown sequence is the reverse of startup to ensure no part is stopped unprepared:

1. Stopping the UI
2. Stopping modules
3. Stopping plugins
4. Stopping Core plugins
5. Stopping the Core

## Loading and Lifecycle Management of Parts

The Core is responsible for loading, initializing, and stopping each part independently. This responsibility is separate from system lifecycle management because each part has its own independent lifecycle.

For each part, the Core goes through these steps:

1. Loading ← Reading the manifest and contract from the file system
2. Validation ← Checking structure and compatibility
3. Initialization ← Injecting configuration and registering in the registry; upon successful registration, the `core:part-loaded` message is published on the system channel
4. Stopping ← Notifying, waiting for operations to complete, removing from the registry; after removal, the `core:part-unloaded` message is published on the system channel

If loading a part encounters an error, the Core ignores that part, logs the error, and continues loading the rest of the parts.

Before initialization, the Core checks for circular dependencies among parts. If a circular dependency is detected, the Core halts initialization and reports a clear error.

## Internal Logging

The Core has a simple internal logging system that is available from the moment of execution. This log has no external dependencies and only prints to the console or standard output.

Internal logging is used in two scenarios:

* Before loading the log plugin — all Core logs are recorded via internal logging
* When the log plugin is unavailable — the Core falls back to internal logging

Once the log plugin is loaded, the Core automatically redirects logs to the plugin.

## Manifest and Contract Validation

Before loading any part, the Core validates its manifest and contract. This validation occurs in two stages:

**Stage One — Structure Validation** (Before Loading):
```text
Do manifest.json and contract.json exist?
Are all mandatory manifest fields present?
Are the versions in the correct format?
Is the part type valid?
Is the architecture version compatible?
Does the contract declared in "implements" exist?
Are essential dependencies available, and do their versions match the version range declared in the manifest?
Do "required" configurations have values?
Are the mandatory contract fields present?
```

**Stage Two — Implementation Validation** (After Loading):
```text
Are all contract methods implemented?
Does the signature of each method match the contract?
```

If the first stage of validation fails, the part is not loaded. If the second stage of validation fails, the part is removed from the registry. In both cases, the Core logs and reports a clear error.

## Service Registry

The Core maintains a central registry where plugins, modules, the platform, and the UI are registered. This registry is the single source of truth for recognizing active parts of the system. The Core uses the registry to manage the lifecycle of the parts.

```text
Register ← A part is registered in the registry after validation
Lookup   ← Checks if a part with this name and contract has been loaded
Remove   ← A part is removed from the registry
```

The registry is solely for managing the lifecycle of the parts. Communication between parts is done exclusively through the channel. No part has direct access to the service or implementation of another part.

## Channels

The Core manages the channels. Each part can have its own separate channel to maintain isolation and security.

In this architecture, channels transfer messages. At the channel level, the architecture does not technically distinguish between an event, request, command, or response. All of these are messages, and their meaning is defined by the message `type` and the contracts of the parts.

### Channel Types

```text
System Channel  ← General messages that all authorized parts can hear
Private Channel ← Private messages of each part declared in the manifest
```

### Channel Operations

```text
Publish     ← A part publishes a message to a channel
Subscribe   ← A part declares that it cares about a specific message type
Unsubscribe ← A part declares that it no longer listens to that message type
```

### Basic Message Structure

Every message must have a `type`. Other fields are added depending on the message's needs. The Core or the channel completes the technical fields when the message is published.

```json
{
  "id": "msg-123",
  "type": "task:created",
  "source": "my-company.module.task",
  "data": {
    "id": "task-1"
  },
  "correlationId": "req-456",
  "timestamp": "2026-06-11T12:00:00.000Z"
}
```

* `id` is the unique identifier of the message and is generated by the channel or the Core.
* `type` is the message type and must be defined in the related contract.
* `source` is the name of the publishing part and is set by the channel or the Core.
* `data` is the message payload and is optional.
* `correlationId` is used to connect multiple messages to the same flow and is optional.
* `timestamp` is the message publish time and is set by the channel or the Core.

### Channel Rules

* A message can include a `data` field.
* Small data can be transferred directly in the message.
* Large data must be kept in a temporary storage plugin, and only its identifier (ID) should be transferred in the message.
* Sensitive data such as passwords or tokens must not be published on the system channel.
* The publisher does not know who receives the message.
* The receiver does not know who published the message.
* No part can directly access another part's channel.
* Only the Core can create a new channel.
* The Core does not publish, filter, or route messages from the parts. The Core's responsibility is solely to create the channels and manage part access to them. Each part publishes its messages directly into the channel it has access to.
* An error in one listener must not prevent message delivery to other listeners.
* Every subscription must be removed when the part stops.

### Matching Related Messages

If a part publishes a message and expects a response or continuation, it must use `correlationId`. All messages related to that flow must include the same `correlationId`.

```text
storage:get        → { correlationId: "req-1", data: { store: "tasks", key: "task-1" } }
storage:get:result → { correlationId: "req-1", data: { value: {...} } }
storage:get:failed → { correlationId: "req-1", data: { code: "STORAGE_NOT_FOUND" } }
```

Managing timeouts, missing responses, and retries is the responsibility of the part that published the original message. The channel may provide helper utilities for this, but those helpers must still use normal messages internally.

### Rules for Creating a Private Channel

A private channel is created only in these cases:

* The part exchanges sensitive data that must not be published on the system channel.
* The part has high-frequency internal messages that are irrelevant to other parts.

The part declares in its manifest that it requires a private channel, and the Core creates it and provides it to the part.

## Error Management

The Core is responsible for containing errors that escape the boundary of any part.

```text
1. The error is contained within the same boundary
2. The Core logs the error
3. The Core notifies via the system channel
4. The system continues operating with degraded capabilities
```

The Core never halts the entire system due to an error in a single part.

The Core's error management functions are private and are not exposed to external parts via the public API. Each part is responsible for managing its own internal errors, and the Core only contains errors that escape a part's boundary.

## Shared State

The Core maintains data that all parts need and that does not have a specific owner.

The Core's shared state is strictly for data that is truly global and independent of any business logic. Any data related to product logic must be managed within a plugin or module.

Examples:

```text
UI Language
Network connection status
```

### Shared State Rules

* Only the Core can mutate the shared state
* The shared state is not exposed to external parts via the Core's public API — state changes are only made from within the Core in response to specific messages
* Parts can read the shared state
* Every change in the shared state is notified via the system channel
* The shared state must be kept to an absolute minimum
* No business data should reside in the Core's shared state

## Configuration

The Core is the access point for general system settings. Configurations are read from the `bootstrap.json` file and made available to all parts.

Important distinction:
* **Product Configuration** — Settings like API addresses and public keys that do not change after startup
* **User Settings** — Things like language and themes, which are managed in the Core's shared state or a separate plugin

### Configuration Rules

* No module or plugin should hardcode configurations
* All configurations must be read from the Core
* Product configuration does not change after system startup
* The Core must not directly access environment variables — the environment source must be injected into the Core from the outside. This principle keeps the Core independent of the execution environment and ensures its testability

## What the Core is Not

To prevent the Core from bloating, the following are explicitly outside the Core's responsibilities:
- Business logic ← Module responsibility
- Infrastructure ← Plugin responsibility
- Data presentation ← UI responsibility
- Execution environment access ← Platform responsibility
- Business data ← Plugin and module responsibility