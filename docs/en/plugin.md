# Plugin

## Introduction

The plugin is the primary unit for implementing shared infrastructure. Plugins provide capabilities that various modules require, but they are not dependent on the logic of any specific module.

Plugins are distinct from modules. Modules implement the product's logic; plugins provide infrastructure. A plugin should not know what product is being built on top of it.

Plugins are part of the system from day one and are initialized before modules. Modules cannot function without their required plugins.

## Plugin Responsibilities

Plugins are responsible for implementing infrastructures that are shared across different projects:

* **Storage** — Data retention and retrieval
* **Authentication** — Managing user identity and access
* **Routing** — Managing navigation between different parts of the product
* **AI Integration** — Connecting to AI models and services
* **Data Analytics** — Collecting and sending analytical messages
* **Notifications** — Sending notifications to the user through various channels

## Structure of a Plugin

Each plugin consists of three parts:

**Manifest** — The file that defines the identity, version, architectural compatibility, implemented contract, dependencies, and required configuration. The Core recognizes and loads the plugin based on the manifest.

**Contract** — The formal agreement specifying what operations this plugin has implemented, what messages it accepts, what messages it publishes, and which channel it uses.

**Internal Implementation** — The execution details of the infrastructure, which is completely private. No part can directly access the internal implementation of a plugin.

## Plugin's Relationship with the Core

### Loading by the Core

The Core recognizes the plugin via its manifest. The information the Core reads from the manifest includes:

* Name and version
* Architecture version
* The implemented contract
* Required and optional dependencies
* Required configuration

After reading the manifest, the Core checks dependencies and compatibility. If any of these are not met, the plugin is not initialized, and the Core reports a clear error.

### Dependency on the Execution Environment

Some plugins only work on specific platforms. The plugin declares this limitation through required dependencies in its manifest. Before initializing the plugin, the Core verifies that the required platform has been loaded.

### Relationship with the Channel

A plugin can interact with the channel in three ways:

**Listening to the System Channel** — The plugin can listen to general system messages.

**Having a Private Channel** — Each plugin can have its own private channel. This channel is strictly for the plugin's internal communications, and no other part can access it directly.

**Publishing Messages on the System Channel** — The plugin can publish messages on the system channel. The access of other parts to the plugin's private channel is defined through the `channels` section in `bootstrap.json`.

### Core Guarantees

The Core provides registered plugins with three guarantees:

* Dependencies declared in the manifest will be available prior to initialization.
* The required configuration will be available.
* During shutdown, sufficient time will be given to complete ongoing operations.

## Plugin Rules

### Communication with Other Parts

* Plugins must not depend on modules.
* Plugins must not depend on each other except through an explicit contract.
* All communication must be conducted through the channel.

### Logic and Responsibility

* A plugin must not contain product logic.
* A plugin must not know which module is using it.
* A plugin must be usable across different projects without modification.
* The plugin's internal implementation must be replaceable without affecting the modules.

### Isolation

* Each plugin must be replaceable without affecting other plugins.
* An error in a plugin must not propagate to modules that do not use it.
* The plugin must contain its internal errors and notify the system by publishing a message on the channel.

## Plugin Lifecycle

Plugins are initialized before modules and stopped after modules.

### Initialization

1. Reading the manifest
2. Checking architecture version compatibility
3. Checking dependencies on other plugins
4. Checking required platform dependencies
5. Loading configuration from the Core
6. Validating the contract implementation
7. Initializing the internal infrastructure
8. Registering in the Core's registry
9. The plugin is ready

### Stopping

1. Unsubscribing from channels
2. Completing ongoing operations
3. Releasing resources
4. Removing from the Core's registry

## What a Plugin is Not

* A plugin is not responsible for product logic; this is the module's responsibility.
* A plugin is not responsible for data presentation; this is the UI's responsibility.
* A plugin must not be designed for a specific project; it must be shareable across different projects.