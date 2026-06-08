# Versioning

## Introduction

Versioning specifies what stage of evolution each part of the system is at and which architecture version it is compatible with. Consistent versioning prevents hidden conflicts, unexpected incompatibilities, and runtime errors.

## Version Formats

### Two-segment — MAJOR.MINOR

For parts that do not have independent bug fixes and whose changes either break structure or add something new:

* **Architecture** — `1.0`, `1.1`, `2.0`
* **Contract** — `1.0`, `1.1`, `2.0`

### Three-segment — MAJOR.MINOR.PATCH

For parts that can have bugs and need independent fixes:

* **Core** — `1.0.0`, `1.0.1`, `1.1.0`
* **Module** — `1.0.0`, `1.0.1`, `1.1.0`
* **Plugin** — `1.0.0`, `1.0.1`, `1.1.0`
* **Platform** — `1.0.0`, `1.0.1`, `1.1.0`
* **UI** — `1.0.0`, `1.0.1`, `1.1.0`

## Meaning of Each Segment

**MAJOR** — A breaking change. Dependent parts must be updated.

**MINOR** — A backward-compatible change that adds new capability. Dependent parts continue to work without modification.

**PATCH** — A bug fix without changing behavior or the contract. Dependent parts continue to work without modification.

## Declaring Compatibility

Every part must explicitly declare which architecture version it is compatible with. This declaration is part of every part's contract and is the basis for compatibility checks by the core.

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

At any moment, only one version of each part exists in the system. The core does not need to manage multiple parallel versions of a part.

This rule specifies three things:

* If a developer wants to use a new version of a part, they must update it in `bootstrap.json`
* All parts that depend on that part must be compatible with the new version
* Verifying and guaranteeing this compatibility is the developer's responsibility, not the core's

If compatibility is not satisfied, the core announces a clear error at startup.

## Versioning Rules

### Changing Versions

* Any change that breaks the contract increments MAJOR and resets MINOR and PATCH to zero
* Any change that adds new capability without breaking compatibility increments MINOR and resets PATCH to zero
* Any bug fix that does not change behavior only increments PATCH

### Compatibility Checks

* The core checks the declared architecture version of every part before loading it
* If a part's architecture version does not match the core's architecture version, that part is not loaded and a clear error is given
* Any core that is compatible with a specific architecture version can load any module, plugin, platform, or UI that declares the same architecture version

### Version Independence

* Every part has its own independent version
* Changing the version of one part does not necessarily change the versions of other parts
* Different parts can evolve at different speeds
* The architecture version and the core version are not necessarily the same
