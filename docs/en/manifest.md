# Manifest

## Introduction

The manifest is a file that every part of the system — plugin, module, platform, and UI — must have. The manifest is the only authoritative source for understanding a part. The core reads the manifest to obtain all the information it needs to load a part, check compatibility, and manage dependencies.

The manifest is different from the contract. The contract specifies what methods a part provides and what messages it publishes. The manifest specifies who a part is, what version it has, what contract it implements, and what it needs.

The core always reads the manifest and contract together. If either is missing, the part is not loaded.

## Format

The manifest is a JSON file named `manifest.json`. The core reads and validates this file before loading any part.

## Structure

Every manifest consists of six sections:

**identity** — The part's identity including name, type, version, description, author, and license.

**compatibility** — The architecture version this part is compatible with.

**implements** — The name and version of the contract this part implements.

**dependencies** — Dependencies are divided into two categories: required dependencies without which the part cannot function, and optional dependencies that are used if available but whose absence does not prevent the part from working.

**config** — The configuration settings this part needs for startup, along with their default values.

**bus** — Declares the need for a private bus. This field is optional and only defined when the part needs a private bus.

## Manifest Validation

The core validates the manifest in two phases before loading any part:

**Phase One — Structure Validation:**
```text
Are all required fields present?
Does the version have the correct format? (MAJOR.MINOR.PATCH)
Is the part type one of the allowed values? (module, plugin, platform, ui)
Does the implements field have a contract name and version?
```

**Phase Two — Content Validation:**
```text
Is the architecture version compatible with the core?
Does the contract declared in implements exist?
Are required dependencies available?
Is the required runtime environment active?
Do required config keys have values?
```

If any of these checks fail, the part is not loaded and the core announces a clear error. Optional dependencies are not checked during validation and their absence is not an error.

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
    "description": "Web UI",
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

Unlike other parts, the core does not have a `manifest.json` file. The core is introduced in `bootstrap.json` and its information is read directly by itself. This exception exists because the core runs before any loading mechanism.

## Rules

* Every part must have a `manifest.json` file
* The manifest must be written before implementation
* The core always reads the manifest and contract together — the absence of either is an error
* Required dependencies must be available before the part starts
* Optional dependencies are used if available and their absence is not an error
* The configuration a part needs is defined only in that part's own manifest
* If a setting has `required: true` and no value is provided for it, the core does not load the part
