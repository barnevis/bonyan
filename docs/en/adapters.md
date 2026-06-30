# Adapters

## Introduction

An adapter is the layer between the application and the outside world. Anything that connects to something external and might differ across platforms or change over time must be managed through an adapter.

The rest of the application never works directly with external technologies. It always works through an adapter. Swapping an adapter only requires a change in the configuration file and no other part of the code changes.

## Core Principle

All adapters that perform the same kind of work must implement exactly the same set of methods. For example, every storage adapter — whether IndexedDB or PostgreSQL — must have the same read, write, delete, and query methods. The rest of the application works only with these methods and does not know which technology is behind them.

The active adapter is declared in the configuration file.

## Adapter Examples

An adapter is written for anything that connects to the outside world and might change. Common examples:

**Storage:** Responsible for saving and retrieving data. Base methods: read, write, delete, and query. Possible implementations: IndexedDB for browser, SQLite for mobile and desktop, PostgreSQL and MariaDB for server.

**Authentication:** Responsible for login, logout, and identity verification. Base methods: login, logout, check login status, and get current user. Possible implementations: email and password, OAuth, biometrics.

**Notifications:** Responsible for showing notifications to the user. The display method differs across platforms but the methods are the same. Possible implementations: browser Notification API, push notifications on mobile via Capacitor, desktop notifications via Tauri.

**File system:** Responsible for reading and writing files. File system access differs across browser, mobile, and desktop but the methods are the same. Possible implementations: File System Access API in browser, Capacitor Filesystem on mobile, Tauri fs on desktop.

**Output:** Responsible for generating output from application data. Possible implementations: PDF, CSV, direct print.

These are examples, not an exhaustive list. Wherever there is a dependency on an external technology, an adapter can be written.

## Relationship Between Adapter and Infrastructure Plugin

An adapter is the implementation of an external technology. An infrastructure plugin hides it behind a service. For example, a storage adapter talks directly to IndexedDB or PostgreSQL, but the storage plugin uses that adapter and registers StorageService in the Registry. Other plugins only work with StorageService and do not know which adapter is behind it.

## Adapter Rules

**Adapters contain no business logic.** An adapter only communicates with the outside world. No rules, calculations, or application-related validation ever enter an adapter.

**An adapter implements all defined methods.** If the method set has five methods, the adapter must have all five.

**An adapter converts its errors to Bonyan's standard error structure.** Raw errors from external technologies must not leak into the rest of the application. Details are in `error.md`.

**An adapter is replaceable in tests.** Because all adapters of the same kind share the same methods, a real adapter can be replaced in tests with a simple in-memory version.