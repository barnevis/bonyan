# Error Management

## Introduction

Error management in this architecture is a contract, not an implementation. Every part must report errors in a specific and predictable way so the core can contain and manage them.

The primary goal is that an error in one part neither stops the system nor gets lost in it.

## Error Types

Errors in this architecture are divided into three categories:

**Startup error** — An error that occurs while loading or starting a part. Examples include a missing dependency, version incompatibility, or incomplete configuration. This error prevents that part from loading but the system continues operating.

**Operational error** — An error that occurs while executing an operation. Examples include a failed method, a dropped connection, or invalid data. This error must be contained at the part's own boundary.

**Critical error** — An error that takes a part completely out of operation. The core removes this part from the registry, notifies via the system bus, and the system continues operating with reduced capability.

## Error Structure

Every error in this architecture must contain the following information:

```json
{
  "type": "operational",
  "source": {
    "part": "my-company.module.task",
    "method": "create"
  },
  "code": "TASK_LIMIT_EXCEEDED",
  "message": "The number of tasks exceeds the allowed limit",
  "timestamp": "2024-01-01T00:00:00Z"
}
```

**type** — The error type: `startup`, `operational`, or `critical`

**source** — The error source including the part name and the method where the error occurred

**code** — A unique error code used for tracking and programmatic handling

**message** — A human-readable message

**timestamp** — The exact time the error occurred

## Error Management Rules

### Each Part's Responsibility

* Every part manages and logs its own internal errors and never exposes a raw error outside its own boundary
* Critical errors that take a part out of operation must be announced via the system bus
* No part may ignore an error

### Core's Responsibility

* The core only logs errors related to startup, loading, and validation of parts
* The core announces critical errors via the system bus
* The core never stops the entire system because of one part's error

### Error Notification

When a part encounters a critical error, the core publishes this event on the system bus:

```json
{
  "event": "core:part-failed",
  "part": "my-company.module.task",
  "type": "critical"
}
```

The UI and other interested parts can listen to this event and respond appropriately.

## What Error Management Is Not

* Error management does not mean hiding errors
* Error management does not mean infinite retries — retry logic must be defined inside the part itself
* Error management is not a substitute for testability
