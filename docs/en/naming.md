# Naming Conventions

## Introduction

Consistent and conventional naming is one of the most critical factors for readability, maintainability, and extensibility in this architecture. When naming conventions are predictable, both humans and AI models can understand the type, owner, and role of each part without needing further explanation.

## General Principles

* All names are written in English.
* Multi-word names use `kebab-case`.
* A dot (`.`) is the separator for the main segments of a name.
* A colon (`:`) separates an event's name from its verb.
* No two parts in a system can have the same name.

## Part Naming Format

All core components of the architecture follow a three-segment format:

```text
vendor.type.name
```

* **Vendor** — The name of the person, team, or organization that wrote this part.
* **Type** — The type of the part within the architecture.
* **Name** — The specific name of this part.

### Core

```text
my-company.core.main
```

The Core is usually unique, but if multiple teams have different cores, the vendor name distinguishes them.

### Module

```text
my-company.module.task
my-company.module.chat
my-company.module.notification
```

### Plugin

```text
my-company.plugin.indexed-db
my-company.plugin.auth
my-company.plugin.router
```

### Platform

```text
my-company.platform.browser
my-company.platform.mobile
my-company.platform.desktop
```

### User Interface (UI)

```text
my-company.ui.web
my-company.ui.mobile
my-company.ui.desktop
```

### Contract

```text
my-company.contract.task
my-company.contract.storage
my-company.contract.auth
```

## Internal Elements Naming

Because internal elements exist within the scope of a specific part, they do not require the vendor name.

### Event Bus

Buses follow the `name.bus` format:

```text
system.bus       ← The system bus, shared among all parts
task.bus         ← The private bus of the task module
indexed-db.bus   ← The private bus of the indexed-db plugin
```

### Events

Events follow the `name:verb` format, which includes two types:

```text
Event (happened) ← Past tense verb     storage:saved
Request          ← Present tense verb  storage:get, storage:save
```

```text
task:created
task:updated
task:deleted
auth:succeeded
auth:failed
storage:synced
storage:saved
storage:save-failed
```

### Methods (Operations)

Methods follow the `name.verb` or `name.verbObject` format:

```text
task.getAll
task.getById
task.create
task.update
task.delete
```

### Core Events

Events that the Core publishes on the system bus follow the `core:verb` format:

```text
core:ready          ← The system is ready
core:part-failed    ← A part encountered a critical error
core:part-loaded    ← A part was successfully loaded
core:part-unloaded  ← A part was removed from the registry
```

## Naming Rules

* The vendor name must be clear and unique to prevent name collisions between different parts.
* The part type must be exactly one of the values: `core`, `module`, `plugin`, `platform`, or `ui`.
* The part name must be short, clear, and descriptive.
* Unknown or ambiguous abbreviations should be avoided.
* An event name must be specific enough to be understood without additional explanation.