# Platform

## Introduction

The platform is the bridge between the product and the runtime environment. The platform's responsibility is to provide unified access to runtime environment capabilities so that the rest of the system does not need to know the details of the runtime environment.

The platform is not part of the product's logic. The platform is only an adaptation layer that allows the same product to run in different environments.

## Platform Responsibilities

The platform is responsible for three things:

**Runtime environment capability access** — Every runtime environment has its own specific capabilities. The platform makes these capabilities available to the rest of the system through unified interfaces — such as access to the filesystem, camera, system notifications, or local storage.

**Application lifecycle management** — Every runtime environment has a different lifecycle. The platform passes this lifecycle to the core so the core can properly manage system startup and shutdown — such as the app opening, going to the background, or closing.

**Adapting to runtime environment constraints** — Every runtime environment has its own specific constraints. The platform hides these constraints from the rest of the system and provides compatible solutions.

## Platform Types

### Browser

A web-based runtime environment that runs in different browsers.

Main capabilities:
* Local storage
* Server communication
* Camera and microphone access with user permission

Main constraints:
* Limited filesystem access
* Dependency on network connection
* Browser security restrictions

### Mobile

A runtime environment based on mobile devices.

Main capabilities:
* Sensor access
* System notifications
* Contacts and calendar access with user permission
* Background execution

Main constraints:
* More limited resources compared to desktop
* Lifecycle management by the operating system
* Background resource access restrictions

### Desktop

A runtime environment based on personal computers.

Main capabilities:
* Full filesystem access
* Parallel process execution
* Operating system integration

Main constraints:
* Differences between operating systems

## Platform Structure

Every platform consists of three parts:

**Manifest** — A file that defines the platform's identity, version, architecture compatibility, implemented contract, required configuration, and platform events. The core reads the manifest to recognize and load the platform.

**Contract** — A formal agreement that specifies what unified interfaces this platform provides. The platform interface must be the same across all runtime environments.

**Internal implementation** — The details of how runtime environment capabilities are accessed, which is completely private.

## Platform Relationship with the Core

The core is the first part to run. The core loads the platform defined in `bootstrap.json`.

### Loading by the Core

After loading the platform, the core receives the following information from it:

* Runtime environment type
* Available capabilities
* Runtime environment constraints

The core uses this information to decide which plugins to load. Plugins that depend on the current runtime environment are only loaded if they are compatible.

### Unified Interface

The platform makes runtime environment capabilities available to the core and plugins through unified interfaces. These interfaces are the same across all platforms even if each platform's internal implementation differs.

## Platform Rules

### Runtime Environment Access

* No part other than the platform can interact directly with the runtime environment
* All access to runtime environment capabilities must go through the platform's interfaces
* The platform must transparently declare capabilities that are not available

### Logic and Responsibility

* The platform must not contain product logic
* The platform must not depend on modules
* The platform must not listen to module events
* The platform interface must be the same across all runtime environments

### Isolation

* Differences between runtime environments must be fully contained inside the platform
* Changing or replacing a platform must not require changes to modules or plugins

## Platform Lifecycle

### Startup

1. Read the manifest
2. Check architecture version compatibility
3. Load configuration from the core
4. Check contract implementation
5. Check available capabilities in the runtime environment
6. Prepare unified interfaces
7. Deliver runtime environment information to the core
8. Platform is ready

### Shutdown

1. The core issues the shutdown command
2. The platform completes operations in progress
3. The platform releases resources

## What a Platform Is Not

* A platform is not responsible for product logic — that is the module's job
* A platform is not responsible for infrastructure — that is the plugin's job
* A platform is not responsible for displaying data — that is the UI's job
