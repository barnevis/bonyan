# Naming

## Introduction

Consistent and conventional naming is one of the most important factors for readability, maintainability, and extensibility of this architecture. When names are predictable, both humans and AI models can understand the type, owner, and role of every part without needing extra explanation.

## General Principles

* All names are written in English
* Multi-word names use `kebab-case`
* A dot separates the main segments of a name
* A colon separates an event name from its verb
* No two parts in a system can have the same name

## Part Naming Format

All main architecture parts follow a three-segment format:

```text
maker.type.name
```

* **maker** — The name of the person, team, or organization that wrote this part
* **type** — The part's type in the architecture
* **name** — The specific name of this part

### Core

```text
my-company.core.main
```

The core is usually unique, but if multiple teams have different cores, the maker name distinguishes them.

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

### UI

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

## Internal Element Naming

Internal elements do not need a maker name since they exist within the scope of a specific part.

### Event Bus

Buses follow the `name.bus` format:

```text
system.bus       ← system bus, shared between all parts
task.bus         ← private bus of the task module
indexed-db.bus   ← private bus of the indexed-db plugin
```

### Events

Events follow the `name:verb` format:

```text
task:created
task:updated
task:deleted
auth:succeeded
auth:failed
storage:synced
```

The event verb must be in the past tense because an event announces what has happened, not what is about to happen.

### Public Interface Methods

Methods follow the `name.verb` or `name.verbObject` format:

```text
task.getAll
task.getById
task.create
task.update
task.delete
```

### Core Events

Events that the core publishes on the system bus follow the `core:verb` format:

```text
core:ready          ← system is ready
core:part-failed    ← a part encountered a critical error
core:part-loaded    ← a part was successfully loaded
core:part-unloaded  ← a part was removed from the registry
```

## Naming Rules

* The maker name must be specific and unique to prevent name conflicts between parts from different sources
* The part type must be exactly one of `core`, `module`, `plugin`, `platform`, or `ui`
* The part name must be short, specific, and descriptive
* Unknown abbreviations must be avoided
* An event name must be specific enough to be understood without extra explanation
