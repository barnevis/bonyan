# Error Management

## Introduction

Error management in this architecture is a contract, not an implementation. Each part must report errors in a specific and predictable manner so that the Core can contain and manage them.

The primary goal is that an error in a single part neither crashes the system nor gets lost within it.

## Types of Errors

Errors in this architecture are divided into three categories:

**Startup Error** — An error that occurs during the loading or initialization of a part. Examples include missing dependencies, version incompatibility, or incomplete configuration. This error prevents that specific part from loading, but the system continues to operate.

**Operational Error** — An error that occurs during the execution of an operation. Examples include a method failure, a dropped connection, or invalid data. This error must be contained within the part's own boundary.

**Critical Error** — An error that completely disables a part. The Core removes this part from the registry, sends a notification via the system bus, and the system continues to operate with degraded capabilities.

## Error Structure

Every error in this architecture must contain this information:

```json
{
  "type": "operational",
  "source": {
    "part": "my-company.module.task",
    "method": "create"
  },
  "code": "TASK_LIMIT_EXCEEDED",
  "message": "Task limit exceeded",
  "timestamp": "2024-01-01T00:00:00Z"
}
```

**type** — The type of error: `startup`, `operational`, or `critical`.

**source** — The source of the error, including the part name and the method where the error occurred.

**code** — A unique error code used for tracking and programmatic management.

**message** — A human-readable message.

**timestamp** — The exact time the error occurred.

## Error Management Rules

### Responsibility of Each Part

* Each part manages and logs its own internal errors and never publishes raw errors outside its boundary.
* Critical errors that cause a part to crash must be reported via the system bus.
* No part should ignore (swallow) an error.

### Core's Responsibility

* The Core only logs errors related to the startup, loading, and validation of parts.
* The Core notifies the system about critical errors via the system bus.
* The Core never halts the entire system due to an error in a single part.

### Error Notification

When a part encounters a critical error, the Core publishes this event on the system bus:

```json
{
  "event": "core:part-failed",
  "part": "my-company.module.task",
  "type": "critical"
}
```

The UI and other interested parts can listen to this event and react appropriately.

## What Error Management is Not

* Error management does not mean hiding errors.
* Error management does not mean infinite retries; retry logic must be defined within the part itself.
* Error management is not a substitute for testability.