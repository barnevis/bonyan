# User Interface (UI)

## Introduction

The User Interface (UI) is the presentation layer of the product. The UI's responsibility is to display data, capture user interaction, collect user-inputted data, and transmit them to the system via the channel.

The UI is the final layer of the architecture. All decisions, rules, and product logic must have been made before reaching the UI. The UI merely displays the result.

## UI Responsibilities

The UI is responsible for four things:

**Data Presentation** — Displays data received via the channel in a comprehensible format for the user. The UI does not interpret the data; it only displays it.

**Capturing User Interaction** — Captures user inputs such as clicks and navigation, and publishes them as messages on the channel.

**Collecting User Data** — Collects data entered by the user, such as forms and text inputs, and transmits them to the system via the channel.

**Sending Messages** — All user interactions and data are transmitted to the system as messages via the channel. The UI does not know which part processes this message.

## Structure of a UI

Each UI consists of three parts:

**Manifest** — The file that defines the identity, version, architectural compatibility, implemented contract, corresponding platform, required configuration, and the need for a private channel. The Core recognizes and loads the UI based on the manifest.

**Contract** — The formal agreement specifying what messages this UI publishes, what messages it receives, and which channels it has access to.

**Internal Implementation** — The details of data presentation and user interaction management, which is completely private.

## Relationship with Other Parts

Data flow in the UI follows a specific cycle:

```text
The UI is initialized
       ↓
The UI publishes the ui:ready message
       ↓
Relevant modules send initial data via the channel
       ↓
The UI receives and displays the data
       ↓
The user interacts
       ↓
The UI publishes a message on the channel
       ↓
The relevant module receives and processes the message
```

### Relationship with Modules

* The UI receives data exclusively through the channel.
* The UI cannot directly call a module's internal logic.
* The UI cannot directly modify a module's data.

### Relationship with the Channel

* All UI communication is conducted through the channel.
* The channel type — public or private — is defined in `bootstrap.json`.
* Sensitive data such as passwords or banking information must not be published on the public channel.
* The UI does not know which part receives its messages.

### Relationship with Plugins

* The UI can use plugins declared in its manifest.
* The UI must never implement business logic.
* Plugins must not depend on the UI or call it directly.

## UI Rules

### Logic and Responsibility

* The UI must not contain business logic.
* The UI must not know the product rules.
* The UI must not decide what data is displayed; this decision is made by the module.
* The UI must not mutate data prior to display; it can only format it.

### Isolation

* The UI must be replaceable without altering the modules.
* Changes in the UI's appearance or technology must not harm the product's logic.
* The UI must not hold the state of product data; data is always received via the channel.

### User Interaction

* Every user interaction must be converted into a specific message.
* Small data can be transferred directly in the message.
* Large data must be kept in a temporary storage plugin, and only its identifier (ID) should be transferred in the message.
* The UI must not predict or simulate the result of an interaction; it waits for the module's response.
* The UI must indicate loading, error, and success states to the user.

## UI Lifecycle

The UI is the last part to be initialized and the first part to be stopped.

### Initialization

1. Reading the manifest
2. Checking architecture version compatibility
3. Checking compatibility with the current platform
4. Loading configuration from the Core
5. Validating the contract implementation
6. Creating the channel based on the definition in `bootstrap.json`
7. Subscribing to required messages on the channels
8. Publishing the `ui:ready` message
9. Receiving initial data via the channel
10. Preparing the initial display
11. The UI is ready

### Stopping

1. The Core issues the command to stop the UI
2. The UI unsubscribes from messages on the channels
3. The UI releases resources

## What a UI is Not

* The UI is not responsible for product logic; this is the module's responsibility.
* The UI is not responsible for maintaining product data; data belongs to the modules.
* The UI is not responsible for infrastructure; this is the plugin's responsibility.
* The UI is not responsible for making decisions; it only displays the results of decisions.