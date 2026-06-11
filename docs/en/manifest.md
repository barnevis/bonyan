# Manifest

## Introduction

The manifest is a file that every part of the system — plugin, module, platform, and UI — must have. The manifest is the single source of truth for identifying a part. From the manifest, the Core obtains all the necessary information for loading, checking compatibility, and managing dependencies.

A manifest is different from a contract. A contract is a formal agreement specifying what operations this part has implemented, what messages it accepts, and what messages it publishes. The manifest specifies who a part is, what version it has, what contract it implements, and what it needs.

The Core always reads the manifest and the contract together. If either is missing, the part is not loaded.

## Manifest Format

The manifest is a JSON file named `manifest.json`. The Core reads and validates this file before loading any part.

## Manifest Structure

Each manifest consists of six sections:

**identity** — The part's identity, including name, type, version, description, author, and license.

**compatibility** — The architecture version with which this part is compatible.

**implements** — The name and version of the contract this part has implemented.

**dependencies** — Dependencies are divided into two categories: `required` dependencies, without which the part will not function, and `optional` dependencies, which are used if available, but their absence does not prevent operation.

**config** — The configurations this part needs for initialization, along with default values.

**bus** — Declaration of the need for a private bus. This field is optional and is defined only if the part requires a private bus.

## Manifest Validation

Before loading any part, the Core validates the manifest in two stages:

**Stage One — Structure Validation:**
```text
Are all mandatory fields present?
Is the version in the correct format? (MAJOR.MINOR.PATCH)
Is the part type one of the allowed values? (module, plugin, platform, ui)
Does the 'implements' field contain the contract name and version?
```

**Stage Two — Content Validation:**
```text
Is the architecture version compatible with the Core?
Does the contract declared in 'implements' exist?
Are the required dependencies available?
Is the required execution environment the current one?
Do the 'required' configurations have values?
```

If any of these checks fail, the part is not loaded, and the Core reports a clear error. Optional dependencies are not checked during validation, and their absence is not an error.

## Plugin Manifest Example

```json
{
  "identity": {
    "name": "my-company.plugin.indexed-db",
    "type": "plugin",
    "version": "2.1.0",
    "description": "IndexedDB-based storage plugin",
    "author": "my-company",
    "license": "MIT"
  },
  "compatibility": {
    "architecture": "1.1"
  },
  "implements": {
    "contract": "my-company.contract.storage",
    "version": "1.0"
  },
  "dependencies": {
    "required": [
      {
        "name": "my-company.platform.browser",
        "version": "1.0.x"
      }
    ],
    "optional": [
      {
        "name": "my-company.plugin.logger",
        "version": "1.x.x"
      }
    ]
  },
  "config": [
    {
      "key": "db-name",
      "required": true,
      "default": null
    },
    {
      "key": "db-version",
      "required": false,
      "default": 1
    },
    {
      "key": "max-size-mb",
      "required": false,
      "default": 50
    }
  ],
  "bus": {
    "private": false
  }
}
```

## Module Manifest Example

```json
{
  "identity": {
    "name": "my-company.module.task",
    "type": "module",
    "version": "1.0.0",
    "description": "Task management module",
    "author": "my-company",
    "license": "MIT"
  },
  "compatibility": {
    "architecture": "1.1"
  },
  "implements": {
    "contract": "my-company.contract.task",
    "version": "1.0"
  },
  "dependencies": {
    "required": [
      {
        "name": "my-company.plugin.indexed-db",
        "version": "2.x.x"
      },
      {
        "name": "my-company.plugin.auth",
        "version": "1.x.x"
      }
    ],
    "optional": [
      {
        "name": "my-company.plugin.logger",
        "version": "1.x.x"
      }
    ]
  },
  "config": [
    {
      "key": "max-tasks-per-user",
      "required": false,
      "default": 1000
    }
  ],
  "bus": {
    "private": false
  }
}
```

## Platform Manifest Example

```json
{
  "identity": {
    "name": "my-company.platform.browser",
    "type": "platform",
    "version": "1.0.0",
    "description": "Browser platform",
    "author": "my-company",
    "license": "MIT"
  },
  "compatibility": {
    "architecture": "1.1"
  },
  "implements": {
    "contract": "my-company.contract.platform",
    "version": "1.0"
  },
  "dependencies": {
    "required": [],
    "optional": []
  },
  "config": [
    {
      "key": "storage-prefix",
      "required": false,
      "default": "app"
    }
  ]
}
```

## UI Manifest Example

```json
{
  "identity": {
    "name": "my-company.ui.web",
    "type": "ui",
    "version": "1.0.0",
    "description": "Web user interface",
    "author": "my-company",
    "license": "MIT"
  },
  "compatibility": {
    "architecture": "1.1"
  },
  "implements": {
    "contract": "my-company.contract.ui",
    "version": "1.0"
  },
  "dependencies": {
    "required": [
      {
        "name": "my-company.platform.browser",
        "version": "1.x.x"
      }
    ],
    "optional": [
      {
        "name": "my-company.plugin.logger",
        "version": "1.x.x"
      }
    ]
  },
  "config": [
    {
      "key": "theme",
      "required": false,
      "default": "light"
    },
    {
      "key": "language",
      "required": false,
      "default": "fa"
    }
  ],
  "bus": {
    "private": true
  }
}
```

## Core Manifest

Unlike other parts, the Core does not have a `manifest.json` file. The Core is declared in `bootstrap.json`, and its information is read directly by itself. This exception is due to the fact that the Core executes before any loading mechanism.

## Manifest Rules

* Every part must have a `manifest.json` file.
* The manifest must be written before implementation.
* The Core always reads the manifest and contract together; the absence of either is an error.
* Required dependencies must be available before the part is initialized.
* Optional dependencies are used if present, and their absence is not an error.
* The configuration required by each part is defined exclusively in that part's manifest.
* If a setting is marked as `required: true` and no value is defined for it, the Core will not load the part.