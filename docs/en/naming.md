# Naming Conventions

## Introduction

Consistent and conventional naming is one of the most critical factors for readability, maintainability, and extensibility in this architecture. When naming conventions are predictable, both humans and AI models can understand the type, owner, and role of each part without needing further explanation.

## General Principles

* All names are written in English.
* Multi-word names use `kebab-case`.
* A dot (`.`) is the separator for the main segments of a name.
* A colon (`:`) separates a message name from its verb and status.
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

### Channel

Channels follow the `name.channel` format:

```text
system.channel       ← The system channel, shared among all parts
task.channel         ← The private channel of the task module
indexed-db.channel   ← The private channel of the indexed-db plugin
```

### Messages

At the channel level, everything is a message. The architecture does not technically distinguish between an event, request, command, or response. The meaning of a message is defined by its name and contract.

Messages follow the `name:verb` format:

```text
name:verb
```

For messages that announce the result or failure of an operation, simple suffixes are used:

```text
name:verb:result  ← Successful result of an operation
name:verb:failed  ← Failure of an operation
```

Examples:

```text
task:create
task:create:result
task:create:failed
task:created
task:updated
task:deleted
storage:get
storage:get:result
storage:get:failed
storage:synced
auth:login
auth:login:result
auth:login:failed
auth:logged-in
```

Recommended readability rule:

```text
Messages that start an operation usually use a present-tense verb: task:create, storage:get
Messages that announce that something happened usually use a past-tense verb: task:created, auth:logged-in
Result and failure messages use :result and :failed
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

### Core Messages

Messages that the Core publishes on the system channel follow the `core:verb` format:

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
* A message name must be specific enough to be understood without additional explanation.