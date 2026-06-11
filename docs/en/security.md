# Security

## Introduction

Security in this architecture is not a separate layer, but a principle that flows through all parts. Each part is responsible for the security of its own boundary, and the Core coordinates the security of the entire system.

The goal of security in this architecture is threefold:
* Prevent unauthorized access between parts
* Prevent the leakage of sensitive data
* Limit the impact of a compromised part on the rest of the system

## Security Principles

### Least Privilege

Each part must only have access to what it needs to perform its duty. Nothing more. This principle is enforced through the manifest. Anything not declared in a part's manifest is unavailable.

### Zero Trust

No part should trust the input of another part without validation. Even if the sender is an internal part of the system.

### Impact Containment

If a part is compromised or exhibits abnormal behavior, its impact must remain confined to that part. The Core's isolation mechanisms guarantee this principle.

### Transparency

Every security action — rejecting a request, filtering an event, or blocking access — must be logged. Hidden security is not reliable security.

## Security Layers

### Layer One — Core Security

The Core is the first and most important line of defense:
* It must only load parts whose manifest and contract are valid.
* It must not register any part without full validation.
* It must create channels strictly based on the project configuration.
* It must control part access to the channels — no part can access a channel that is not defined in the configuration.
* It must log any attempt at unauthorized access to a channel.

### Layer Two — Event Bus Security

Channels are the only way to communicate and transfer data between parts:
* Small data can be transferred directly in the message.
* Large data must be kept in a temporary storage plugin, and only its identifier (ID) should be transferred in the message.
* Sensitive data like passwords, tokens, or personal information must not be published via the system bus.
* Messages containing sensitive data must use a private bus.

### Layer Three — Boundary Security

Each part is responsible for the security of its own boundary:
* Received inputs must be validated before processing.
* No part should trust raw external data.
* Outbound outputs must not leak internal system information or raw error details.

### Layer Four — Configuration Security

* Sensitive keys like API keys, database passwords, and service tokens must not be stored as plain text in `bootstrap.json`.
* These values must be provided via environment variables or a secret management service.
* No part should log sensitive keys.

## Security Responsibilities

```text
Core                  ← Controlling access to channels, manifest and contract validation
Authentication Plugin ← Managing identity and tokens
Modules               ← Input validation, protecting business logic
Platform              ← Execution environment security
UI                    ← Preventing the display of sensitive data
```

## What Security is Not

* Security is not a separate plugin that can be added or removed.
* Security is not the responsibility of just one part.
* Security should not be an excuse to complicate the architecture; simple and predictable rules are more secure.