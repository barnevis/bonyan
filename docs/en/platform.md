# Platform

## Introduction

The platform is the bridge between the product and the execution environment. The platform's responsibility is to provide unified access to the execution environment's capabilities, so that the rest of the system's parts do not need to know the details of the execution environment.

The platform is not part of the product's logic. The platform is merely an adaptation layer that allows the same product to run in different environments.

## Platform Responsibilities

The platform is responsible for three things:

**Access to Execution Environment Capabilities** — Every execution environment has its own specific capabilities. The platform exposes these capabilities to the rest of the system through unified interfaces. Examples include access to the file system, camera, system notifications, or local storage.

**Application Lifecycle Management** — Every execution environment has a different lifecycle. The platform relays this lifecycle to the Core so the Core can properly manage the startup and shutdown of the system. Examples include the application opening, going to the background, or closing.

**Adapting to Environment Limitations** — Every execution environment has its own constraints. The platform hides these limitations from the rest of the system and provides compatible solutions.

## Types of Platforms

### Browser

A web-based execution environment that runs in various browsers.

Core Capabilities:
* Local storage
* Server communication
* Access to camera and microphone with user permission

Core Limitations:
* Limited file system access
* Dependency on network connection
* Browser security constraints

### Mobile

An execution environment based on mobile devices.

Core Capabilities:
* Access to sensors
* System notifications
* Access to contacts and calendar with user permission
* Background execution

Core Limitations:
* More limited resources compared to desktop
* Lifecycle management by the operating system
* Constraints on background resource access

### Desktop

An execution environment based on personal computers.

Core Capabilities:
* Full file system access
* Parallel process execution
* Operating system integration

Core Limitations:
* Differences between various operating systems

## Structure of a Platform

Each platform consists of three parts:

**Manifest** — The file that defines the platform's identity, version, architectural compatibility, implemented contract, required configuration, and messages. The Core recognizes and loads the platform based on the manifest.

**Contract** — The formal agreement specifying what operations this platform has implemented, what messages it accepts, what messages it publishes, and what execution environment capabilities it provides.

**Internal Implementation** — The details of accessing the execution environment's capabilities, which is completely private.

## Platform's Relationship with the Core

The Core is the first part to execute. The Core loads the platform defined in `bootstrap.json`.

### Loading by the Core

After loading the platform, the Core receives the following information from it:

* Type of execution environment
* Available capabilities
* Execution environment limitations

The Core uses this information to make decisions about loading plugins. Plugins dependent on the current execution environment are only loaded if they are compatible.

### Unified Interface

The platform provides the execution environment's capabilities to the Core and plugins through unified interfaces. These interfaces are identical across all platforms, even if the internal implementation of each platform differs.

## Platform Rules

### Access to the Execution Environment

* No part other than the platform can interact directly with the execution environment.
* All access to the execution environment's capabilities must be conducted through the platform's interfaces.
* The platform must transparently declare capabilities that are unavailable.

### Logic and Responsibility

* The platform must not contain product logic.
* The platform must not depend on modules.
* The platform must not listen to module messages.
* The platform's interface must be identical across all execution environments.

### Isolation

* Differences between execution environments must be completely contained within the platform.
* Changing or replacing a platform must not require modifications to modules or plugins.

## Platform Lifecycle

### Initialization

1. Reading the manifest
2. Checking architecture version compatibility
3. Loading configuration from the Core
4. Validating the contract implementation
5. Checking available capabilities in the execution environment
6. Preparing unified interfaces
7. Delivering execution environment information to the Core
8. The platform is ready

### Stopping

1. The Core issues the stop command
2. The platform completes ongoing operations
3. The platform releases resources

## What a Platform is Not

* The platform is not responsible for product logic; this is the module's responsibility.
* The platform is not responsible for infrastructure; this is the plugin's responsibility.
* The platform is not responsible for data presentation; this is the UI's responsibility.