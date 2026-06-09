# Plugin

## Introduction

A plugin is the primary unit for implementing shared infrastructure. Plugins provide capabilities that different modules need but are not dependent on any specific module's logic.

Plugins are distinct from modules. Modules implement product logic; plugins provide infrastructure. A plugin must not know what product is built on top of it.

Plugins are part of the system from day one and are started before modules. Modules cannot function without the plugins they depend on.

## Plugin Responsibilities

Plugins are responsible for implementing infrastructure that is shared across different projects:

* **Storage** — persisting and retrieving data
* **Authentication** — managing user identity and access
* **Routing** — managing navigation between different parts of the product
* **AI integration** — connecting to AI models and services
* **Analytics** — collecting and sending analytical events
* **Notifications** — sending notifications to users through different channels

## Plugin Structure

Every plugin consists of three parts:

**Manifest** — A file that defines the plugin's identity, version, architecture compatibility, implemented contract, dependencies, and required configuration. The core reads the manifest to recognize and load the plugin.

**Contract** — A formal agreement that specifies what public methods this plugin provides. The plugin's contract must be stable because multiple modules depend on it.

**Internal implementation** — The details of how the infrastructure is implemented, which is completely private. No other part can directly access a plugin's internal implementation.

## Plugin Relationship with the Core

### Loading by the Core

The core recognizes a plugin from its manifest. Information the core reads from the manifest:

* Name and version
* Architecture version
* The contract it implements
* Required and optional dependencies
* Required configuration

After reading the manifest, the core checks dependencies and compatibility. If any are not satisfied, the plugin is not started and the core announces a clear error.

### Runtime Environment Dependency

Some plugins only work on specific platforms. A plugin declares this constraint through required dependencies in its manifest. The core verifies that the required platform is loaded before starting the plugin.

### Relationship with the Event Bus

A plugin can interact with the event bus in three ways:

**Listening to the system bus** — A plugin can listen to public system events.

**Having a private bus** — Every plugin can have its own private bus. This bus is only for the plugin's internal communications and no other part can directly access it.

**Publishing messages on the system bus** — A plugin can publish messages on the system bus. Access by other parts to a plugin's private bus is defined through the `channels` section in `bootstrap.json`.

### Core Guarantees

The core gives three guarantees to registered plugins:

* Dependencies declared in the manifest will be available before startup
* Required configuration will be available
* At shutdown, sufficient time will be given to complete operations in progress

## Plugin Rules

### Communication with Other Parts

* Plugins must not depend on modules
* Plugins must not depend on each other except through an explicit contract
* All communication must happen through the event bus

### Logic and Responsibility

* A plugin must not contain product logic
* A plugin must not know which module uses it
* A plugin must be usable across different projects without modification
* A plugin's internal implementation must be replaceable without affecting modules

### Isolation

* Every plugin must be replaceable without affecting other plugins
* An error in one plugin must not propagate to modules that do not use it
* A plugin must contain its own internal errors and notify by publishing a message on the event bus

## Plugin Lifecycle

Plugins are started before modules and stopped after modules.

### Startup

1. Read the manifest
2. Check architecture version compatibility
3. Check dependencies on other plugins
4. Check required dependencies on the platform
5. Load configuration from the core
6. Check contract implementation
7. Start internal infrastructure
8. Register in the core registry
9. Plugin is ready

### Shutdown

1. Unsubscribe from event buses
2. Complete operations in progress
3. Release resources
4. Remove from the core registry

## What a Plugin Is Not

* A plugin is not responsible for product logic — that is the module's job
* A plugin is not responsible for displaying data — that is the UI's job
* A plugin must not be designed for a specific project — it must be shareable across different projects
