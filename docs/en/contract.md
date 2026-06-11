# Contract

## Introduction

A contract is a formal agreement specifying what operations a part has implemented, what messages it accepts, what messages it publishes, and which channel it uses. The operations of a part are not directly accessible and can only be interacted with through the event bus. The contract is the common language among the Core, plugins, modules, the platform, and the user interface.

A contract is different from a manifest. A manifest specifies who a part is and what it needs. A contract specifies what a part provides and how it communicates.

Without a contract, each part would have to be aware of the internal details of another part. With a contract, each part only needs to know what the other party has promised.

## Contract Format

The contract is a JSON file named `contract.json`. The Core reads and validates this file simultaneously with the manifest.

## Contract Structure

Each contract consists of four sections:

**Identity (`identity`)** — Includes the name, description, and version of the contract. The name is the unique identifier of this contract in the system. No two contracts can have the same name.

**Methods (`methods`)** — The list of operations implemented by this part. Each operation specifies what this part can do, what parameters it accepts, what output it returns, and what errors might occur. These operations are not directly callable. Access to them is only possible through channel messages.

**Messages (`messages`)** — The list of messages this part publishes. Each message must have a specific type, description, data structure, and emission time (when it is sent).

**Suggested Channel (`suggestedChannel`)** — The identifier of the channel suggested for use with this part's messages.

## Contract Validation

The Core performs contract validation in two stages:

**Stage One — Structure Validation** (Before loading the part):
```text
Are the identity, methods, and messages fields present?
Is the contract name unique?
Is the version in the correct format? (MAJOR.MINOR)
Does each method have a name, parameters, and a return type?
Does each message have a type, data structure, and emission time?
Is the suggested channel specified?
```

**Stage Two — Implementation Validation** (After loading the part):
```text
Are all declared methods implemented?
Does the signature of each method match the contract?
```

If structure validation fails, the part is not loaded. If implementation validation fails, the part is removed from the registry, and the Core reports a clear error.

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
      "description": "Get a stored value",
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
      "when": "After successful initialization"
    },
    {
      "type": "storage:synced",
      "description": "Data synchronized",
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
      "description": "Get all tasks",
      "params": [],
      "returns": "Task[]",
      "errors": ["TASK_FETCH_FAILED"]
    },
    {
      "name": "getById",
      "description": "Get a specific task",
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
      "description": "New task created",
      "data": { "id": "string" },
      "when": "After successful creation"
    },
    {
      "type": "task:updated",
      "description": "Task updated",
      "data": { "id": "string" },
      "when": "After successful update"
    },
    {
      "type": "task:deleted",
      "description": "Task deleted",
      "data": { "id": "string" },
      "when": "After successful deletion"
    }
  ],
  "suggestedChannel": "task.bus"
}
```

## Contract Rules

### Defining a Contract

* Every module, plugin, platform, and UI must have a `contract.json` file.
* The contract must be written before implementation.
* The contract belongs to the part that defines it, not the part that uses it.
* The Core always reads the contract and manifest together; the absence of either is an error.

### Messages and Data

* Messages can include a `data` field.
* Small data can be transferred directly in the message.
* Large data must be kept in a temporary storage plugin, and only its identifier (ID) should be transferred in the message. Determining whether data is "large" is up to the developer; the general principle is that if transferring data via a message causes performance issues, it should be changed to the storage and ID transfer method.
* Sensitive data like tokens or passwords must not be included in the message.
* The suggested channel is merely a suggestion, and the developer can change it in the project configuration.

### Modifying a Contract

* A change that adds a new method or message without altering existing ones increments the minor version.
* A change that breaks the signature of an existing method or alters a message's data structure increments the major version.
* No part can modify another part's contract.

## What a Contract is Not

* A contract does not include implementation details.
* A contract does not explain *how* something is done, only *what* is done.
* A contract does not include dependencies or configuration; these are defined in the manifest.