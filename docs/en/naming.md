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

In Bonyan there are two types of events that must be distinguished from each other.

### Domain Event

An event published by a plugin to announce that something has happened in the system. These events are written in the past tense and follow this format:

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

### UI Action Event

An event published by the UI layer to announce that the user has made a lightweight change in the UI, such as changing a filter or opening a panel. This event is not a substitute for a direct service call and is never used to trigger core product logic such as saving or deleting; those operations always go through a direct service call. These events start with the `ui:` prefix and use a noun form for the action:

```
ui:action-requested
```

Examples:
- `ui:filter-changed`
- `ui:panel-opened`
- `ui:view-mode-changed`

The key difference is that a domain event says "this happened" and a UI action event says "a lightweight UI request or state change has occurred". A UI action event never replaces a direct service call for a core product operation.

Like domain events, UI action events carry only identifying data, not the full payload, and must never carry sensitive data such as passwords or tokens.

## General Naming Rules

**Lowercase and hyphens.** All names are written in lowercase. If a name has multiple words, hyphens are used. Example: `myfinance.budget-reports`.

**Descriptive names.** A name should say what the plugin or event is, not how it works.

**Abbreviations are not allowed.** `transactions` is correct, `txn` is not. Readability matters more than brevity.

**Domain events are in the past tense.** A domain event is something that has already happened. `saved` is correct, `save` is not. The exception is Core events that start with `core:`.

**UI action events start with `ui:`.** All events published by the UI layer start with `ui:` and end with `-changed`, `-opened`, or similar patterns that describe a UI state change, not the execution of a core product operation.