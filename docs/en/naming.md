# Naming

## Introduction

Naming conventions in Bonyan are few and simple. The goal is that anyone reading the code immediately understands what they are looking at and where it came from.

## Plugin Naming

All plugins follow a two-segment format:

```
vendor.name
```

**vendor** is the name of the person, team, or organization that wrote the plugin. This segment prevents name collisions between plugins from different sources.

**name** is a short, descriptive name for the plugin. It should convey what the plugin does.

Examples:
- `myfinance.transactions`
- `myfinance.accounts`
- `myfinance.storage`
- `acme.reports`

No two plugins in the same system may share the same name. The Core checks this uniqueness at startup.

## Service Naming

Services take the name of the plugin that provides them, plus a `.service` suffix:

```
vendor.name.service
```

Examples:
- `myfinance.transactions.service`
- `myfinance.storage.service`
- `acme.reports.service`

## Event Naming

Events follow this format:

```
pluginname:eventname
```

**pluginname** is the second segment of the plugin name, without the vendor.

**eventname** describes what happened, not what should happen.

Examples:
- `transactions:saved`
- `transactions:deleted`
- `accounts:updated`
- `storage:ready`
- `core:plugin-failed`
- `core:plugin-crashed`

## General Naming Rules

**Lowercase and hyphens.** All names are written in lowercase. If a name has multiple words, hyphens are used. Example: `myfinance.budget-reports`.

**Descriptive names.** A name should say what the plugin or event is, not how it works.

**Abbreviations are not allowed.** `transactions` is correct, `txn` is not. Readability matters more than brevity.

**Event names are in the past tense.** An event is something that has already happened. `saved` is correct, `save` is not. The exception is Core events that start with `core:`.