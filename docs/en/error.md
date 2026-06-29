# Error Management

## Introduction

Errors are inevitable in any system. Bonyan defines a uniform structure for all errors so that they are predictable, loggable, and manageable. This structure is defined by the Core and every part of the system must follow it.

## Error Structure

Every error in Bonyan must include the following fields:

**code:** A unique code for this type of error. The code allows the error to be identified without depending on the message text.

**message:** A human-readable description of the error. This message is for logging and debugging.

**source:** The name of the plugin or part where the error occurred. Follows the `vendor.name` format.

**type:** The error type. One of three values: `startup`, `operational`, or `critical`.

**timestamp:** The exact time the error occurred.

**detail:** Additional information useful for debugging. This field is optional.

## Error Types

### Startup Error

Occurs while loading a plugin. This type of error is handled by the Core.

Common causes: invalid manifest, incompatible architecture version, missing required dependency, duplicate name.

Core behavior: the plugin is deactivated, the error is recorded in the internal log, and the `core:plugin-failed` event is published via the Event Bus. Startup of the remaining plugins continues unless the failing plugin is an infrastructure plugin.

### Operational Error

Occurs during a normal operation. This type of error is handled by the plugin itself.

Common causes: invalid data, failed operation, unavailable resource.

Plugin behavior: the plugin handles the error, notifies the user if needed, and execution continues. This type of error must not leave the plugin's boundary.

### Critical Error

Completely disables a plugin and is unrecoverable. The plugin reports this error to the Core.

Common causes: loss of connection to a required service, corruption of base data, an unexpected and unrecoverable failure.

Core behavior: the plugin is deactivated, the error is recorded in the internal log, and the `core:plugin-crashed` event is published via the Event Bus. The system continues with degraded capability.

## Error Management Responsibilities

**The Core** handles startup errors and critical errors reported by plugins.

**Plugins** handle their own operational errors. If an error is critical, the plugin reports it to the Core.

**Adapters** receive raw errors from external technologies and convert them to Bonyan's standard error structure. Raw errors never leak into the rest of the application.

## Error Events

The Core publishes these events via the Event Bus:

**`core:plugin-failed`** when a plugin encounters an error during startup.

**`core:plugin-crashed`** when a plugin reports a critical error.

Any plugin that cares about these events can listen and respond appropriately. For example, the UI layer can listen to `core:plugin-crashed` and display a suitable message to the user.

## Error Management Rules

**Every error must have the standard structure.** An error without structure cannot be logged or managed.

**Every part manages and logs its own internal errors.** Raw errors never leave a part's boundary. Adapters, plugins, and services are each responsible for converting their errors to Bonyan's standard structure.

**Operational errors do not leave the plugin's boundary.** The plugin is responsible for managing its own operational errors.

**Critical errors must be announced via the Event Bus.** A plugin cannot hide a critical error. The Core receives it, deactivates the plugin, and publishes the appropriate event.

**No part may ignore an error.** An ignored error will eventually lead to unpredictable behavior.

**The Core does not stop the entire system because of a failure in one non-infrastructure part.** The system continues with degraded capability.

**Error management does not mean hiding errors.** An error must be logged and announced when necessary. Hiding errors makes debugging impossible.

**Error management does not mean infinite retries.** If retry logic is needed, it must be defined within the part itself with a fixed and configurable count.

**Error management is not a substitute for testing.** Having error management does not justify skipping tests. These two tools complement each other, they do not replace each other.

**Error messages are for humans, error codes are for the system.** Logic never makes decisions based on message text. Decisions are made based on error codes.

**Sensitive data must not appear in errors.** Passwords, tokens, and personal information must not be placed in the error structure, not even in the detail field.