# Security

## Introduction

Security in this architecture is not a separate layer — it is a principle that runs through every part. Each part is responsible for the security of its own boundary and the core coordinates the security of the entire system.

The security goals of this architecture are three:
* Prevent unauthorized access between parts
* Prevent sensitive data from leaking
* Limit the impact of a compromised part on the rest of the system

## Security Principles

### Least Privilege

Every part must only have access to what it needs to perform its function — nothing more. This principle is enforced through the manifest. Anything not declared in a part's manifest is not accessible.

### Zero Trust

No part must trust input from another part without validation — even if the sender is an internal part of the system.

### Blast Radius Containment

If a part is compromised or behaves abnormally, its impact must remain contained to that part. The core's isolation mechanisms guarantee this principle.

### Transparency

Every security action — rejecting a request, filtering an event, or blocking access — must be logged. Hidden security is not trustworthy security.

## Security Layers

### Layer One — Core Security

The core is the first and most important line of defense:

* Only load parts whose manifest and contract are valid
* Never register a part in the registry without complete validation
* Control access to the registry — every access request is checked against the dependencies declared in the manifest
* Log every rejected access request with the requester's name and the requested service

### Layer Two — Registry Security

The registry does not give all parts free access:

```text
All parts  ←  can only access parts they have declared in their manifest
```

If a part requests access to a service it has not declared in its manifest, the core rejects the request and logs the incident.

### Layer Three — Event Bus Security

The event bus is a communication channel that can carry data:

* Small data can be transferred directly in the message
* Large data must be stored in a temporary storage plugin and only its identifier passed in the message
* Sensitive data such as passwords, tokens, or personal information must not be published on the system bus
* Messages containing sensitive data must use a private channel

### Layer Four — Boundary Security

Each part is responsible for the security of its own boundary:

* Received inputs must be validated before processing
* No part must trust raw external data
* Sent outputs must not expose internal system information or raw error details

### Layer Five — Configuration Security

* Sensitive keys such as API keys, database passwords, and service tokens must not be stored as plain text in `bootstrap.json`
* These values must be supplied through environment variables or a secret management service
* No part must log sensitive keys

## Security Responsibilities

```text
Core              ← registry access control, manifest and contract validation
Auth plugin       ← identity and token management
Modules           ← input validation, business logic protection
Platform          ← runtime environment security
UI                ← preventing display of sensitive data
```

## What Security Is Not

* Security is not a separate plugin that can be added or removed
* Security is not the sole responsibility of one part
* Security must not be an excuse to complicate the architecture — simple and predictable rules are the most secure
