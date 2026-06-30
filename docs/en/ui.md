# UI Layer

## Introduction

The UI layer is responsible for displaying data to the user and receiving interaction from them. The UI has no knowledge of product logic, business rules, or infrastructure. This separation ensures that changes to the appearance or technology of the UI do not affect product logic, and the UI can be replaced without modifying any plugin.

The UI is not a plugin but, like plugins, it has a manifest and an entry point that the Core reads at startup.

## UI Manifest

The UI manifest is part of Bonyan's architecture because it defines the UI's access boundary. The Core grants access only to the services declared in the UI manifest and provides nothing else to the UI.

The UI manifest declares which services the UI needs and which events it publishes. Events declared here must be lightweight and limited to UI-level changes, not core product operations. Its structure is similar to a plugin manifest but lighter:

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
      "name": "ui:filter-changed",
      "description": "The user has changed the display filter",
      "data": [
        { "name": "filterId", "type": "string" }
      ]
    },
    {
      "name": "ui:panel-opened",
      "description": "The user has opened a panel",
      "data": [
        { "name": "panelId", "type": "string" }
      ]
    }
  ]
}
```

The UI manifest has no `provides` section because the UI does not provide services. It has no `type` section because it is not a plugin.

## Manifest and Entry Point

Like plugins, the UI has a manifest and an entry point. At startup, the Core gives the UI's entry point the same three things it gives plugins: the services declared in the UI manifest's `dependencies`, the ability to publish and listen to events via the Event Bus, and the UI's final configuration.

The exact file names, folder structure, and function signatures are implementation details and belong in the best-practices document, not in this one.

When the UI publishes an event, the `source` field is taken from the UI's name in its manifest — a value like `myapp.ui` — not from anything determined at runtime.

## The Primary User Action Path

This path defines exactly how data and control flow between the UI, a service, and a plugin when the user performs an action. This primary path always goes through a direct service call, not the Event Bus.

**For reading data:** The UI calls the service declared in its manifest directly and retrieves data for display.

**For sending a user request:** The UI sends the data the user entered directly to the relevant method on the declared service. The core logic always executes inside the plugin's service, even when the UI calls that service directly. The UI never executes any logic itself.

**After the operation completes:** The plugin that provides the service publishes a domain event once the operation succeeds, so the UI or other parts can learn about the change. Publishing the domain event is the plugin's responsibility, not the UI's.

**What role the Event Bus plays here:** The Event Bus is for coordination and notification, not the primary path for executing a user operation. Operations such as saving or deleting always go through a direct service call. A UI action event is only used to announce lightweight UI changes, such as a filter change or a panel being opened. It is not a substitute for calling a service directly and is never used to transfer a complete form payload or to trigger core logic.

## UI Responsibilities

The UI layer has four responsibilities:

**Displaying data:** To display data, the UI calls the relevant service directly. The UI may format data for display but does not modify the underlying data.

**Receiving user interaction:** Handles clicks, scrolls, selections, and other user interactions.

**Receiving data from the user:** Collects data the user enters and sends it directly to the relevant service.

**Publishing UI action events:** Only to announce lightweight UI changes, the UI publishes the appropriate event with the `ui:` prefix. Like every event in Bonyan, this event carries only identifying data — never a full payload and never sensitive data. The full payload must always be transferred through a direct service call, not through an event.

## What the UI Is Not

**The UI is not responsible for product logic.** Business rules, calculations, and product-related validation live in plugins, even when the UI calls a plugin's service directly.

**The UI does not own product data.** Data belongs to product plugins.

**The UI does not decide what data is valid.** The product plugin decides what data is valid and presentable. The UI decides how that data is displayed.

**The UI is not responsible for infrastructure.** Storage, routing, and other base services do not belong to the UI layer.

**The UI does not know product rules.** Those rules live in product plugins.

## The Display and Logic Boundary

**Allowed:** Display formatting such as localizing a date, showing a number with thousand separators, or changing color based on a value. Also allowed is visual sorting or filtering that only changes how data is presented, not the underlying data itself.

**Not allowed:** Real data filtering, calculations, validation of product rules, or any decision that relates to business logic.

## Implementation Recommendation

Bonyan does not impose any specific technology for the UI layer. The choice of technology is the developer's.

That said, Web Components are recommended. Web Components are a browser standard, depend on no framework, and work without modification in Capacitor and Tauri.