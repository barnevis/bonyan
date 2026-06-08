# Contract

## Introduction

A contract is a formal agreement that specifies what methods a part of the system provides, what messages it publishes, and which channel it uses. The contract is the shared language between the core, plugins, modules, the platform, and the UI.

The contract is different from the manifest. The manifest specifies who a part is and what it needs. The contract specifies what a part provides and how it communicates.

Without a contract, every part must know the internal details of every other part. With a contract, each part only needs to know what the other side has promised.

## Format

The contract is a JSON file named `contract.json`. The core reads and validates this file at the same time as the manifest.

## Structure

Every contract consists of four sections:

**identity** — Includes the contract's name, description, and version. The name is the unique identifier of this contract in the system. No two contracts can have the same name.

**methods** — The list of public methods this part provides. Every method must have a name, description, input parameters, return type, and possible errors.

**messages** — The list of messages this part publishes. Every message must have a type, description, data structure, and a specified time of publication.

**suggestedChannel** — The identifier of the channel recommended for this part's messages.

## Contract Validation

The core validates the contract in two phases:

**Phase One — Structure Validation** (before loading the part):
```text
Are the identity, methods, and messages fields present?
Is the contract name unique?
Does the version have the correct format? (MAJOR.MINOR)
Does every method have a name, parameters, and return type?
Does every message have a type, data structure, and publication time?
Is the suggested channel specified?
```

**Phase Two — Implementation Validation** (after loading the part):
```text
Are all declared methods implemented?
Does each method's signature match the contract?
```

If structure validation fails, the part is not loaded. If implementation validation fails, the part is removed from the registry and the core announces a clear error.

## Plugin Contract Example

```json
{
  "identity": {
    "name": "my-company.contract.storage",
    "description": "General storage contract",
    "version": "1.0"
  },
  "methods": [
    {
      "name": "get",
      "description": "Retrieve a stored value",
      "params": [
        { "name": "key", "type": "string" }
      ],
      "returns": "any",
      "errors": ["STORAGE_NOT_FOUND", "STORAGE_FAILED"]
    },
    {
      "name": "set",
      "description": "Store a value",
      "params": [
        { "name": "key", "type": "string" },
        { "name": "value", "type": "any" }
      ],
      "returns": "boolean",
      "errors": ["STORAGE_FAILED"]
    },
    {
      "name": "delete",
      "description": "Delete a stored value",
      "params": [
        { "name": "key", "type": "string" }
      ],
      "returns": "boolean",
      "errors": ["STORAGE_FAILED"]
    }
  ],
  "messages": [
    {
      "type": "storage:ready",
      "description": "Plugin is ready to use",
      "data": {},
      "when": "After successful startup"
    },
    {
      "type": "storage:synced",
      "description": "Data has been synchronized",
      "data": { "keys": "string[]" },
      "when": "After every write operation"
    },
    {
      "type": "storage:failed",
      "description": "Storage operation failed",
      "data": { "code": "string" },
      "when": "When an error occurs"
    }
  ],
  "suggestedChannel": "system.bus"
}
```

## Module Contract Example

```json
{
  "identity": {
    "name": "my-company.contract.task",
    "description": "Task management contract",
    "version": "1.0"
  },
  "methods": [
    {
      "name": "getAll",
      "description": "Retrieve all tasks",
      "params": [],
      "returns": "Task[]",
      "errors": ["TASK_FETCH_FAILED"]
    },
    {
      "name": "getById",
      "description": "Retrieve a specific task",
      "params": [
        { "name": "id", "type": "string" }
      ],
      "returns": "Task",
      "errors": ["TASK_NOT_FOUND"]
    },
    {
      "name": "create",
      "description": "Create a new task",
      "params": [
        { "name": "data", "type": "TaskInput" }
      ],
      "returns": "Task",
      "errors": ["TASK_VALIDATION_FAILED", "TASK_LIMIT_EXCEEDED"]
    },
    {
      "name": "update",
      "description": "Update a task",
      "params": [
        { "name": "id", "type": "string" },
        { "name": "data", "type": "TaskInput" }
      ],
      "returns": "Task",
      "errors": ["TASK_NOT_FOUND", "TASK_VALIDATION_FAILED"]
    },
    {
      "name": "delete",
      "description": "Delete a task",
      "params": [
        { "name": "id", "type": "string" }
      ],
      "returns": "boolean",
      "errors": ["TASK_NOT_FOUND"]
    }
  ],
  "messages": [
    {
      "type": "task:created",
      "description": "A new task was created",
      "data": { "id": "string" },
      "when": "After successful creation"
    },
    {
      "type": "task:updated",
      "description": "A task was updated",
      "data": { "id": "string" },
      "when": "After successful update"
    },
    {
      "type": "task:deleted",
      "description": "A task was deleted",
      "data": { "id": "string" },
      "when": "After successful deletion"
    }
  ],
  "suggestedChannel": "task.bus"
}
```

## Contract Rules

### Defining a Contract

* Every module, plugin, platform, and UI must have a `contract.json` file
* The contract must be written before implementation
* The contract belongs to the part that defines it, not the part that uses it
* The core always reads the contract and manifest together — the absence of either is an error

### Messages and Data

* Messages can carry a `data` field
* Small data can be transferred directly in the message
* Large data must be stored in a temporary storage plugin and only its identifier passed in the message. Determining what counts as large data is left to the developer; the general principle is that if transferring data through a message causes a performance problem, it should switch to the storage-and-identifier approach.
* Sensitive data such as tokens or passwords must not be placed in messages
* The suggested channel is only a suggestion — the developer can change it in the project configuration

### Changing a Contract

* A change that adds a new method or message without modifying existing ones increments the minor version
* A change that breaks an existing method's signature or modifies a message's data structure increments the major version
* No part can change another part's contract

## What a Contract Is Not

* A contract does not include implementation details
* A contract does not explain how something is done — only what is done
* A contract does not include dependencies or configuration — these are defined in the manifest
