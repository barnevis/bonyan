# Core

## Introduction

The Core is the smallest and most stable part of Bonyan. Its sole responsibility is to make the rest of the system work together. The Core contains no business logic, provides no services, and knows nothing about the content of application data.

The Core is the first thing to start and the last thing to shut down.

The Core never stops the entire system because of a failure in one part.

## Responsibilities

### Registry

The Registry holds the address of every registered service. When a plugin registers a service, the Registry stores its address. When another plugin needs that service, the Registry returns the address.

The Registry knows where each service is. It does not know what a service does and plays no role in calling it.

### Event Bus

The Event Bus is Bonyan's one-way announcement system. A part publishes an event to inform others that something has happened. The publisher does not wait for a response and does not know who is listening.

**Event Bus rules:**

Events are for announcements only. A plugin announces that something happened. If a listener needs full data, it must request it from the relevant service.

A plugin sends only the event type and the minimum identifying information. Before publishing, the Core adds a unique event ID, the name of the publishing plugin, and a timestamp. This ensures the source cannot be forged and the timestamp comes from a single point.

Sensitive data such as passwords, tokens, and personal information must never be published with an event.

A listener only reads an event and never modifies it.

Event names follow the format `pluginname:eventname`. The full event structure is described in `services.md`.

### Plugin Lifecycle

The lifecycle manages the startup and shutdown sequence of plugins. Plugins are either active or inactive. There is no intermediate state.

### System Lifecycle

The Core is responsible for the controlled startup and shutdown of the entire system.

**Startup steps:**

1. The Core reads the configuration file.
2. Adapters are prepared based on the configuration.
3. Infrastructure plugins start in the order defined in the configuration. Each infrastructure plugin announces readiness after successfully registering its service.
4. Application plugins start in the order defined in the configuration.
5. After all plugins have started, the `core:ready` event is published.

**Shutdown steps:**

1. The `core:shutdown` event is published so all parts have a chance to prepare.
2. Application plugins shut down in reverse startup order.
3. Infrastructure plugins shut down in reverse startup order.
4. The Core shuts down.

### Startup Validation

Before activating any plugin, the Core reads its manifest and checks three things:

- The declared architecture version is compatible with the Core's version.
- All required dependencies of the plugin exist in the Registry.
- The plugin name is unique in the system.

If any of these checks fail, the plugin is deactivated and the Core logs the reason. Startup of the remaining plugins continues unless the failing plugin is an infrastructure plugin.

After successful validation, the Core passes the service object directly to the plugin. This means the plugin receives a direct reference to the service and can call its methods without any intermediary. From that point on, the Core plays no role in those service calls.

### Internal Logging

The Core logs its own operations. This log covers plugin startup and shutdown, the result of each plugin's validation, system errors, and unauthorized service access attempts.

This log belongs to the Core. No plugin has direct access to it. If the application needs general logging, a separate plugin must be written for that purpose.

### Error Structure and System Error Management

The Core defines the error structure that all parts of the system must follow. Full details are in `error.md`.

There are three types of errors:

**Startup error** occurs while loading a plugin. The Core handles this: the plugin is deactivated, the `core:plugin-failed` event is published, and startup continues.

**Operational error** occurs during a normal operation. The plugin handles this itself and it must not leak outside the plugin's boundary.

**Critical error** completely disables a plugin. The plugin reports this to the Core. The Core deactivates the plugin, publishes the `core:plugin-crashed` event, and the system continues with degraded capability.

### Internal State Management

The Core maintains a simple internal state necessary to do its job:

- The list of registered plugins and the status of each (active or inactive)
- The list of services registered in the Registry and the address of each

This state belongs to the Core. No plugin has direct access to it. Application state — business data — never enters the Core.

### Configuration

The Core reads the project configuration file as its first startup step. This file is the only source from which the Core learns which adapters and plugins to start and in what order.

**Configuration rules from the Core's perspective:**

The Core has no hardcoded plugins or adapters. Everything comes from the configuration file.

If the configuration file is missing or invalid, the Core does not start.

The Core follows the order defined in the configuration exactly. The developer is responsible for the correctness of that order.

Full details of the configuration file structure are in `configuration.md`.

## Core Boundaries

### What the Core is

The Core is a communication infrastructure. Its job is to let plugins find each other, listen to events, and have their lifecycle managed.

### What the Core is not

**The Core does not provide services.** StorageService, RouterService, NotificationService, and StateService are all provided by infrastructure plugins, not by the Core. The Core only holds their addresses.

**The Core has no business logic.** No rules, calculations, or application-related validation ever enter the Core.

**The Core is not a communication intermediary.** When a plugin gets a service address from the Registry, the call happens directly between the plugin and the service. The Core is not in between.

**The Core is not a general logger.** The Core's internal log covers only the Core's own operations. Application logging must be handled by a separate plugin.

**The Core does not hold application state.** Business data and UI state belong to plugins, not the Core.

## Core Security

The Core is the first and most important line of defense in the system.

The Core only loads plugins that have a valid manifest. Any attempt to access a service not declared in the manifest is rejected and logged. The Core restricts each plugin's access to only the services declared in that plugin's manifest.

Full details are in `security.md`.