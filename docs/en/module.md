# Module

## Introduction

The module is the primary unit for implementing the product's logic and features. Each module represents an independent business capability that has a distinct boundary and assumes a single responsibility.

Modules are distinct from plugins. Plugins provide infrastructure; modules implement the product's logic.

## Module Responsibilities

Modules are responsible for implementing four things:

* **Business Capabilities** — What the product does for the user.
* **Workflows** — How each capability is executed from start to finish.
* **System Behaviors** — What happens in response to messages.
* **Product Rules** — The constraints and conditions that the product's logic must adhere to.

## Structure of a Module

Each module consists of three parts:

**Manifest** — The file that defines the module's identity, version, architectural compatibility, implemented contract, dependencies, configuration, and messages. The Core recognizes and loads the module based on the manifest.

**Contract** — The formal agreement specifying what operations this module has implemented, what messages it publishes, what messages it accepts, and which channel it uses.

**Internal Logic** — The implementation of the module's capabilities, which is completely private and cannot be accessed directly by any other part.

## Module Characteristics

Each module must:

* Have a specific responsibility
* Have a clear boundary
* Have limited dependencies
* Have an independent structure
* Be independently developable
* Have predictable behavior

Modules must not:

* Create hidden dependencies
* Mutate the state of other modules
* Depend on the internal details of other parts

## Module Rules

### Communication with Other Parts

* Modules cannot directly depend on the internal implementation of another module.
* Modules communicate with each other and other parts exclusively through the event bus.
* Modules can use plugins, but not the internal logic of other modules.
* All communication must be conducted through explicit contracts.

### Data Ownership

* Each module is the exclusive owner of its own data.
* No part can directly modify a module's data.
* Requests for data and requests for data modification must be made via the bus.

### Logic and Responsibility

* Each module must have a clear and single responsibility.
* A module's logic must not depend on the type of UI, execution environment, or database.
* A module should not know who receives its messages.

### Error Isolation

* An error in a module must not propagate to the rest of the system.
* A module must contain its internal errors and notify the system via the event bus.

## Module Lifecycle

### Initialization

The Core follows this sequence to initialize each module:

1. Reading the manifest
2. Checking architecture version compatibility
3. Checking dependencies
4. Loading configuration from the Core
5. Validating the contract implementation
6. Connecting to required plugins
7. Initializing the internal logic
8. Registering in the Core's registry
9. The module is ready

If any of these steps encounter an error, the module is not initialized, and the Core reports a clear error.

### Stopping

1. Unsubscribing from messages
2. Completing ongoing operations
3. Releasing resources
4. Removing from the Core's registry

## What a Module is Not

* A module is not responsible for data presentation; this is the UI's responsibility.
* A module is not responsible for providing infrastructure; this is a plugin's responsibility.
* A module must not contain the logic of more than one business capability.