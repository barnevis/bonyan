# Versioning

## Introduction

Versioning specifies at what stage of evolution each part of the system is and which version of the architecture it is compatible with. Consistent versioning prevents hidden conflicts, unexpected incompatibilities, and runtime errors.

## Versioning Format

### Two-Part — MAJOR.MINOR

For parts that do not have independent bug fixes and whose changes either break the structure or add something new:

* **Architecture** — `1.0`, `1.1`, `2.0`
* **Contract** — `1.0`, `1.1`, `2.0`

### Three-Part — MAJOR.MINOR.PATCH

For parts that can contain bugs and require independent fixes:

* **Core** — `1.0.0`, `1.0.1`, `1.1.0`
* **Module** — `1.0.0`, `1.0.1`, `1.1.0`
* **Plugin** — `1.0.0`, `1.0.1`, `1.1.0`
* **Platform** — `1.0.0`, `1.0.1`, `1.1.0`
* **UI** — `1.0.0`, `1.0.1`, `1.1.0`

## Meaning of Each Segment

**MAJOR** — A change that breaks compatibility. Dependent parts must be updated.

**MINOR** — A change that maintains compatibility and adds new functionality. Dependent parts work without modification.

**PATCH** — A bug fix without altering behavior or the contract. Dependent parts work without modification.

## Declaring Compatibility

Each part must explicitly declare which version of the architecture it is compatible with. This declaration is part of each part's contract and forms the basis for compatibility checking by the Core.

### Core

```text
my-company.core.main
version: 1.2.0
architecture: 1.1
```

### Module

```text
my-company.module.task
version: 1.0.0
architecture: 1.1
```

### Plugin

```text
my-company.plugin.indexed-db
version: 2.1.0
architecture: 1.1
```

### Platform

```text
my-company.platform.browser
version: 1.0.0
architecture: 1.1
```

### UI

```text
my-company.ui.web
version: 1.0.0
architecture: 1.1
```

## One Version at a Time

At any given moment, there is only one version of each part in the system. The Core does not need to manage multiple parallel versions of a single part.

This rule specifies three things:

* If a developer wants to use a new version of a part, they must update it in `bootstrap.json`.
* All parts that depend on that part must be compatible with the new version.
* Checking and ensuring this compatibility is the responsibility of the developer, not the Core.

If compatibility is not met, the Core reports a clear error during startup.

## Versioning Rules

### Version Changes

* Any change that breaks the contract increments the MAJOR version and resets MINOR and PATCH to zero.
* Any change that adds a new feature without breaking compatibility increments the MINOR version and resets PATCH to zero.
* Any bug fix that does not alter behavior only increments the PATCH version.

### Compatibility Checking

* Before initializing any part, the Core checks the architecture version declared by that part.
* If a part's architecture version does not match the Core's architecture version, that part is not initialized, and a clear error is thrown.
* Any Core compatible with a specific architecture version can load any module, plugin, platform, or UI that declares the same architecture version.

### Version Independence

* Each part has its own independent version.
* Changing the version of one part does not necessarily change the versions of other parts.
* Different parts can evolve at different speeds.
* The architecture version and the Core version are not necessarily the same.