# UI

## Introduction

The UI is the product's display layer. The UI's responsibility is to display data, receive user interaction, collect data from the user, and transfer them to the system through the event bus.

The UI is the last layer of the architecture. All decisions, rules, and product logic must have been made before reaching the UI. The UI only displays the result.

## UI Responsibilities

The UI is responsible for four things:

**Displaying data** — Displays data received through the event bus in a form that is understandable to the user. The UI does not interpret data — it only displays it.

**Receiving user interaction** — Receives user inputs such as clicks and navigation and publishes them as messages on the event bus.

**Collecting data from the user** — Collects data the user enters such as forms and text inputs, and transfers them to the system through the event bus.

**Sending messages** — All user interactions and data are transferred to the system as messages through the event bus. The UI does not know which part processes these messages.

## UI Structure

Every UI consists of three parts:

**Manifest** — A file that defines the UI's identity, version, architecture compatibility, implemented contract, corresponding platform, required configuration, and need for a private bus. The core reads the manifest to recognize and load the UI.

**Contract** — A formal agreement that specifies what interfaces this UI provides for receiving data, what messages it publishes, and what channel it suggests.

**Internal implementation** — The details of displaying data and managing user interactions, which is completely private.

## Relationship with Other Parts

The data flow in the UI follows a defined cycle:

```text
UI starts up
       ↓
UI publishes ui:ready message
       ↓
Relevant modules send initial data through the event bus
       ↓
UI receives and displays the data
       ↓
User interacts
       ↓
UI publishes the message on the event bus
       ↓
The relevant module receives and processes the message
```

### Relationship with Modules

* The UI receives data only through the event bus
* The UI cannot directly call a module's internal logic
* The UI cannot directly modify a module's data

### Relationship with the Event Bus

* All UI communication happens through the event bus
* The bus type — public or private — is defined in `bootstrap.json`
* Sensitive data such as passwords or banking information must not be published on the public bus
* The UI does not know which part receives its messages

### Relationship with Plugins

* The UI can use plugins it has declared in its manifest
* The UI must never implement business logic
* Plugins must not depend on the UI or call it directly

## UI Rules

### Logic and Responsibility

* The UI must not contain business logic
* The UI must not know product rules
* The UI must not decide what data to display — this decision is made by the module
* The UI must not modify data before displaying it — it can only format it

### Isolation

* The UI must be replaceable without changing modules
* Changes in the UI's appearance or technology must not damage product logic
* The UI must not hold product data state — data is always received through the event bus

### User Interaction

* Every user interaction must be converted into a specific message
* Small data can be transferred directly in the message
* Large data must be stored in a temporary storage plugin and only its identifier passed in the message
* The UI must not predict or simulate the result of an interaction — it waits for the module's response
* The UI must show loading, error, and success states to the user

## UI Lifecycle

The UI is the last part to start and the first part to stop.

### Startup

1. Read the manifest
2. Check architecture version compatibility
3. Check compatibility with the current platform
4. Load configuration from the core
5. Check contract implementation
6. Create buses based on the definition in `bootstrap.json`
7. Subscribe to required messages on the buses
8. Publish `ui:ready` message
9. Receive initial data through the event bus
10. Prepare the initial display
11. UI is ready

### Shutdown

1. The core issues the UI shutdown command
2. The UI unsubscribes from messages on the buses
3. The UI releases resources

## What the UI Is Not

* The UI is not responsible for product logic — that is the module's job
* The UI is not responsible for maintaining product data — data belongs to modules
* The UI is not responsible for infrastructure — that is the plugin's job
* The UI is not responsible for making decisions — it only displays the result of decisions
