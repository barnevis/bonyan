# UI Layer

## Introduction

The UI layer is responsible for displaying data to the user and receiving interaction from them. The UI has no knowledge of product logic, business rules, or infrastructure. This separation ensures that changes to the appearance or technology of the UI do not affect product logic, and the UI can be replaced without modifying any plugin.

The UI is not a plugin but, like plugins, it has a manifest that the Core reads at startup.

## UI Manifest

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
        { "name": "formData", "type": "TransactionInput" }
      ]
    },
    {
      "name": "ui:filter-changed",
      "description": "The user has changed the display filter",
      "data": [
        { "name": "filter", "type": "FilterInput" }
      ]
    }
  ]
}
```

The UI manifest has no `provides` section because the UI does not provide services. It has no `type` section because it is not a plugin.

## Folder Structure

The UI lives in its own folder. The Core expects two specific files in the root of that folder:

**manifest.json:** The UI manifest.

**index.js:** The UI entry point through which the Core passes the required services.

## index.js Contract

`index.js` exports an object with a defined structure:

```js
export default {
  start(dependencies) {
    // The Core passes the declared services here
    // The UI initializes
  },
  stop() {
    // Clean up resources
  }
}
```

## UI Responsibilities

The UI layer has four responsibilities:

**Displaying data:** To display data, the UI calls the relevant service directly. The UI may format data for display but does not modify the underlying data.

**Receiving user interaction:** Handles clicks, scrolls, selections, and other user interactions.

**Receiving data from the user:** Collects data the user enters, such as filling out a form.

**Publishing UI action events:** When the user performs an action, the UI publishes the appropriate event with the `ui:` prefix. The relevant plugin listens and handles the processing.

## What the UI Is Not

**The UI is not responsible for product logic.** Business rules, calculations, and product-related validation live in plugins.

**The UI does not own product data.** Data belongs to product plugins.

**The UI does not decide what data to display.** That decision belongs to the product plugin.

**The UI is not responsible for infrastructure.** Storage, routing, and other base services do not belong to the UI layer.

**The UI does not know product rules.** Those rules live in product plugins.

## Data Flow

Data in Bonyan flows in two directions:

**From service to UI:** To display data, the UI calls the relevant service directly. For example, to show a list of transactions, it calls TransactionService and renders the received data.

**From UI to plugin:** The user performs an action. The UI publishes a UI action event. The plugin listens, processes the data, and publishes a domain event. The UI can listen to the domain event and fetch updated data from the service.

The UI never directly modifies the state of a plugin.

## The Display and Logic Boundary

**Allowed:** Display formatting such as localizing a date, showing a number with thousand separators, or changing color based on a value. Also allowed is visual sorting or filtering that only changes how data is presented, not the underlying data itself.

**Not allowed:** Real data filtering, calculations, validation of product rules, or any decision that relates to business logic.

## Implementation Recommendation

Bonyan does not impose any specific technology for the UI layer. The choice of technology is the developer's.

That said, Web Components are recommended. Web Components are a browser standard, depend on no framework, and work without modification in Capacitor and Tauri.