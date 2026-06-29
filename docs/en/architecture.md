# Architecture

## Introduction

Bonyan is a layered, plugin-based architecture for building web applications using Vanilla JS, HTML, and CSS. It has no dependency on any framework, library, or build tool.

The primary goals of Bonyan are:

- Complete separation of business logic from the user interface
- Extensibility without modifying the core
- Ability to run on browser, mobile (Capacitor), and desktop (Tauri) without any changes to the application code

## Foundational Principles

These principles guide every design decision in Bonyan. Whenever there is ambiguity, the correct answer is the one that aligns with these principles.

### Simplicity over complexity

If two solutions solve the same problem, the simpler one is correct. Complexity must be justified.

### Boundaries are precise and documented

Every part knows what it does and what it does not do. Ambiguity in responsibilities is the source of most architectural bugs.

### Business logic must be independent

Logic knows nothing about the user interface, the platform, or the storage technology. This independence is what makes testing, migration, and reuse possible.

### The Core must stay small and unaware

The Core gives addresses, delivers messages, and manages lifecycle. Business logic, display rules, and storage details never enter the Core.

### Portability is the default

Every part should be movable to a new project. Hidden dependencies destroy this ability.

### The developer is responsible

The Core executes without judgment. Responsibility for following the architecture's conventions rests with the developer. This preserves simplicity and avoids unnecessary overhead.

## Overview Diagram

```
┌─────────────────────────────────────────────────┐
│                   UI Layer                       │
│            Web Components · CSS Variables        │
├─────────────────────────────────────────────────┤
│                    Plugins                       │
│    Infrastructure · Product · Feature · Platform │
├──────────────────┬──────────────────────────────┤
│    Services      │          Adapters             │
│  Base · Plugin   │  Storage · Auth · Filesystem  │
├──────────────────┴──────────────────────────────┤
│                     Core                         │
│  Registry · Event Bus · Lifecycle · Validation   │
└─────────────────────────────────────────────────┘
```

## Layers

Bonyan consists of five layers. Each layer has a single, well-defined responsibility and no layer reaches into another layer's internals.

### Layer One — Core

The smallest and most stable part of the system. The Core contains no business logic. Its sole responsibility is to make the rest of the system work together.

The Core has seven responsibilities:

- **Registry:** Holds the address of every registered service. It knows where each service is, not what it does.
- **Event Bus:** A one-way messaging system. A part publishes a message and does not wait for a response.
- **Lifecycle:** Manages the startup and shutdown sequence of plugins.
- **Startup Validation:** Before activating any plugin, the Core reads its manifest and verifies that the architecture version is compatible, required dependencies exist, and the plugin name is unique.
- **Internal Logging:** The Core logs its own operations — plugin startup, validation errors, unauthorized access attempts. This log is internal and no plugin has direct access to it.
- **Error Structure and System Error Management:** The Core defines the error structure that all parts of the system must follow. It also manages system-level errors: when a plugin reports a critical error, the Core deactivates it, announces the failure via the Event Bus, and the system continues with degraded capability.
- **Internal State Management:** The Core maintains a simple internal state: the list of registered plugins and their status, and the list of services available in the registry. This state belongs to the Core and no plugin has direct access to it.

### Layer Two — Services

Services are capabilities that all plugins can use. All services are provided by plugins, not by the Core. The difference lies only in when they are registered:

**Base services** are provided by infrastructure plugins that must start before all others. Examples: StorageService, RouterService, NotificationService, StateService. Their startup order is defined in the configuration file.

**Application services** are registered by regular plugins during startup so that other plugins can use them. Example: TransactionService registered by the transactions plugin and used by the reports plugin.

In both cases, a plugin asks the Registry for the address of a service, then calls that service directly. The Core is not involved in the call itself.

### Layer Three — Plugins

Every feature of the application is implemented as a plugin. Every plugin has one implementation and one manifest. The architecture does not concern itself with the internal structure of a plugin. What matters is that the plugin declares itself through its manifest and communicates with the rest of the system through the Registry and the Event Bus.

Each plugin type serves a distinct purpose:

- **Infrastructure:** Provides base services that other plugins depend on.
- **Product:** Implements the core business logic of the application.
- **Feature:** Provides optional capabilities the application can work without.
- **Platform:** Handles compatibility with the runtime environment.
- **Transport:** Manages message transfer between the system and the outside world.

The user interface does not live inside a plugin. It belongs entirely to the UI layer.

### Layer Four — UI Layer

The UI layer is completely separate from plugins. It displays data and forwards user interactions to the logic layer. No business logic exists here.

The UI layer is responsible for four things: displaying data, receiving user interaction, receiving data from the user, and publishing events when appropriate.

The UI layer never holds product data, never applies business rules, and never decides what data to display. Those decisions belong to product plugins.

### Layer Five — Adapters

An adapter is written for anything that connects to the outside world and might differ across platforms or change over time.

All adapters of the same kind implement an identical set of methods. The rest of the application does not know or care which adapter is active. Swapping an adapter only requires a change in the configuration file.

Examples of what adapters cover: storage (IndexedDB, PostgreSQL, MariaDB), authentication (email and password, OAuth, biometrics), notifications (browser, mobile push, desktop), file system (browser File API, Capacitor Filesystem, Tauri fs), and output (PDF, CSV, print).

## Communication Rules

Two communication mechanisms exist in Bonyan. The boundary between them is strict and must not be blurred.

**Service** is used when a part needs something and expects a response. The call is direct: the plugin gets the address from the Registry and calls the service.

**Event Bus** is used when a part announces that something happened and does not wait for a response. Any part that needs to know listens independently.

Plugins do not know each other directly. A plugin asks the Registry for the address of the service it needs and then calls that service directly. The Registry is a discovery point, not a communication intermediary.

Sensitive data such as passwords, tokens, and personal information must not be published via the Event Bus.

## Testability Principles

Bonyan's architecture is designed so that each part can be tested independently.

**Plugin logic is testable without the Core.** Because logic has no dependency on the Core, the UI, or storage, it can be tested directly without starting the whole system.

**Adapters are replaceable in tests.** Because all adapters of the same kind share one set of methods, a real adapter can be replaced in tests with a simple in-memory version.

**Services are simulatable.** Every service is accessible via the Registry, so in tests a fake service can be registered and the plugin's behavior can be verified without depending on the real service.

**The UI is testable independently of logic.** Because UI components only receive data and emit events, they can be tested with synthetic data.