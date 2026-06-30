# UI Layer

## Introduction

The UI layer is responsible for displaying data to the user and receiving interaction from them. The UI has no knowledge of product logic, business rules, or infrastructure. This separation ensures that changes to the appearance or technology of the UI do not affect product logic, and the UI can be replaced without modifying any plugin.

The UI is not a plugin but, like plugins, it has a manifest that the Core reads at startup.

## UI Manifest

The UI manifest is part of Bonyan's architecture because it defines the UI's access boundary. The Core grants access only to the services declared in the UI manifest and provides nothing else to the UI.

The UI manifest declares which services the UI needs and which events it publishes. Its structure is similar to a plugin manifest but lighter:

```json
{
  "name": "myapp.ui",
  "version": "1.0.0",
  "dependencies": {
    "required": [
      "myapp.transactions.service",
      "myapp.accounts.service"
    ],
    "optional": [
      "myapp.reports.service"
    ]
  },
  "events": [
    {
      "name": "ui:save-transaction-requested",
      "description": "The user has requested to save a transaction",
      "data": [
        { "name": "requestId", "type": "string" }
      ]
    },
    {
      "name": "ui:filter-changed",
      "description": "The user has changed the display filter",
      "data": [
        { "name": "filterId", "type": "string" }
      ]
    }
  ]
}
```

The UI manifest has no `provides` section because the UI does not provide services. It has no `type` section because it is not a plugin.

## Entry Point

At startup, the Core gives the UI's entry point access to the same three things it gives plugins: the services declared in the UI manifest's `dependencies`, the ability to publish and listen to events via the Event Bus, and the UI's final configuration.

The exact shape of this exchange, including file names and function signatures, is an implementation detail and belongs in the best-practices document, not in this one.

## The Primary User Action Path

This path defines exactly how data and control flow between the UI, a service, and a plugin when the user performs an action.

**For reading data:** The UI calls the service declared in its manifest directly and retrieves data for display.

**For sending a user request:** The UI sends the data the user entered directly to the relevant method on the declared service. The core logic always executes inside the plugin's service, even when the UI calls that service directly. The UI never executes any logic itself.

**After the operation completes:** The plugin that provides the service publishes a domain event once the operation succeeds, so the UI or other parts can learn about the change. Publishing the domain event is the plugin's responsibility, not the UI's.

**When a UI action event is used:** A UI action event is only for announcing lightweight UI changes or requests, such as a filter change or the opening of a form. It is not a substitute for calling a service directly and is not used to transfer a complete form payload or to trigger core logic.

## UI Responsibilities

The UI layer has four responsibilities:

**Displaying data:** To display data, the UI calls the relevant service directly. The UI may format data for display but does not modify the underlying data.

**Receiving user interaction:** Handles clicks, scrolls, selections, and other user interactions.

**Receiving data from the user:** Collects data the user enters and sends it directly to the relevant service.

**Publishing UI action events:** To announce lightweight UI changes or requests, the UI publishes the appropriate event with the `ui:` prefix. Like every event in Bonyan, this event carries only identifying data — never a full payload and never sensitive data. If an event carries an identifier, the full data associated with that identifier must already be stored in a declared service so the listener can retrieve it.

## What the UI Is Not

**The UI is not responsible for product logic.** Business rules, calculations, and product-related validation live in plugins, even when the UI calls a plugin's service directly.

**The UI does not own product data.** Data belongs to product plugins.

**The UI does not decide on product rules or what data is valid.** What data is valid, what rules govern it, and what should be displayed are decisions the product plugin makes.

**The UI is not responsible for infrastructure.** Storage, routing, and other base services do not belong to the UI layer.

**The UI does not know product rules.** Those rules live in product plugins.

## The Display and Logic Boundary

**Allowed:** Display formatting such as localizing a date, showing a number with thousand separators, or changing color based on a value. Also allowed is visual sorting or filtering that only changes how data is presented, not the underlying data itself.

**Not allowed:** Real data filtering, calculations, validation of product rules, or any decision that relates to business logic.

## Implementation Recommendation

Bonyan does not impose any specific technology for the UI layer. The choice of technology is the developer's.

That said, Web Components are recommended. Web Components are a browser standard, depend on no framework, and work without modification in Capacitor and Tauri.