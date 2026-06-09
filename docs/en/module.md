# Module

## Introduction

A module is the primary unit for implementing product logic and capabilities. Each module represents an independent business capability with a clear boundary and a single responsibility.

Modules are distinct from plugins. Plugins provide infrastructure; modules implement product logic.

## Module Responsibilities

Modules are responsible for implementing four things:

* **Business capabilities** — what the product does for the user
* **Workflows** — how each capability is executed from start to finish
* **System behaviors** — what happens in response to events
* **Product rules** — the constraints and conditions that product logic must respect

## Module Structure

Every module consists of three parts:

**Manifest** — A file that defines the module's identity, version, architecture compatibility, implemented contract, dependencies, configuration, and events. The core reads the manifest to recognize and load the module.

**Contract** — A formal agreement that specifies what public methods this module provides. The contract is the only part of the module that the rest of the system knows about.

**Internal logic** — The implementation of the module's capabilities, which is completely private. No other part can directly access it.

## Module Characteristics

Every module must:

* Have a clear responsibility
* Have explicit boundaries
* Have limited dependencies
* Have an independent structure
* Be independently developable
* Have predictable behavior

Modules must not:

* Create hidden dependencies
* Modify the state of other modules
* Depend on the internal details of other parts

## Module Rules

### Communication with Other Parts

* Modules cannot depend directly on another module's internal implementation
* Modules communicate with each other and other parts only through the event bus
* Modules can use plugins, but not the internal logic of other modules
* All communication must happen through explicit contracts

### Data Ownership

* Each module is the exclusive owner of its own data
* No part can directly modify a module's data
* Requests to read or change data must go through the event bus using the message types declared in the contract

### Logic and Responsibility

* Every module must have one clear and single responsibility
* A module's logic must not depend on the UI type, runtime environment, or database
* A module must not know who receives its events

### Error Isolation

* An error in one module must not propagate to the rest of the system
* A module must contain its own internal errors and notify via the event bus

## Module Lifecycle

### Startup

The core follows this sequence to start every module:

1. Read the manifest
2. Check architecture version compatibility
3. Check dependencies
4. Load configuration from the core
5. Check contract implementation
6. Connect to required plugins
7. Start internal logic
8. Register in the core registry
9. Module is ready

If any of these steps fail, the module is not started and the core announces a clear error.

### Shutdown

1. Unsubscribe from events
2. Complete operations in progress
3. Release resources
4. Remove from the core registry

## What a Module Is Not

* A module is not responsible for displaying data — that is the UI's job
* A module is not responsible for providing infrastructure — that is the plugin's job
* A module must not contain logic for more than one business capability
