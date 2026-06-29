# Security

## Introduction

Security in Bonyan is not an add-on feature. It is built into the architecture from the start. Every plugin has access only to what it has explicitly declared, and the Core enforces these boundaries.

Bonyan's security philosophy is simple: anything not declared is forbidden.

## Declaring Access

Every plugin must explicitly declare in its manifest which services it needs. This declaration is made in the `dependencies` section of the manifest. The Core provides access only to these services at startup.

## Security Enforcement by the Core

The Core is the first and most important line of defense in the system.

**At startup:** The Core validates every plugin's manifest. A plugin with an invalid manifest is not started. The Core verifies that all services declared in `dependencies` are available and passes the service object directly to the plugin. This means the plugin receives a direct reference to the service and can call its methods without any intermediary. From that point on, the Core plays no role in those calls.

**Attempting to access an undeclared service:** If a plugin requests a service at runtime that it did not declare in its manifest, the request is rejected and recorded in the internal log.

## Sensitive Data

Sensitive data includes passwords, authentication tokens, encryption keys, and personal user information.

Rules for sensitive data:

**Sensitive data is not published via the Event Bus.** The Event Bus is public and all plugins can listen to it.

**Sensitive data is not placed in the error structure.** Not even in the detail field.

**Sensitive data is only transferred through declared services.** And only to plugins that have declared the necessary access.

## Adapter Security

Adapters communicate with the outside world and are sensitive points in the system.

A storage adapter must not store raw data without processing. If data requires encryption, this responsibility must be explicitly defined.

An authentication adapter is responsible for securely holding login credentials. This information is never transferred to other parts of the system.

## Security Logging

The Core records these security events in its internal log:

- An attempt to access an undeclared service
- An attempt to access a service that does not exist
- A plugin that was started with an invalid manifest

This log belongs to the Core and no plugin has direct access to it.

## Security Responsibilities

**The Core** enforces access boundaries and logs security events.

**Plugins** declare only the access they actually need. Declaring more access than necessary is a security weakness.

**Adapters** properly handle sensitive data coming from the outside world and never let it leak out.

**The developer** is responsible for the correctness of manifest declarations. The Core enforces what is declared, not what is correct.

## Security Rules

**Anything not declared is forbidden.** This is the foundational security principle in Bonyan and has no exceptions.

**Declare the minimum access needed.** A plugin declares only the access it genuinely requires. Excess access expands the attack surface.

**Sensitive data only through secure paths.** Passwords, tokens, and personal information are never transferred via the Event Bus or the error structure.

**Security is not the same as complexity.** Simple and firm rules are better than complex systems that no one implements correctly.