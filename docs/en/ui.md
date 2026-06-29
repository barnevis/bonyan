# UI Layer

## Introduction

The UI layer is responsible for displaying data to the user and receiving interaction from them. The UI has no knowledge of product logic, business rules, or infrastructure. This separation ensures that changes to the appearance or technology of the UI do not affect product logic, and the UI can be replaced without modifying any plugin.

## UI Responsibilities

The UI layer has four responsibilities:

**Displaying data:** Renders data received from product plugins. The UI may format data for display — for example showing a date in a localized format or a number with thousand separators — but it does not modify the underlying data.

**Receiving user interaction:** Handles clicks, scrolls, selections, and other user interactions.

**Receiving data from the user:** Collects data the user enters, such as filling out a form, and sends it to the relevant plugin for processing.

**Publishing events:** When the user performs an action, the UI publishes the appropriate event. The relevant plugin listens and handles the processing.

## What the UI Is Not

**The UI is not responsible for product logic.** Business rules, calculations, and product-related validation live in plugins, not in the UI.

**The UI does not own product data.** Data belongs to product plugins. The UI displays it but does not hold it.

**The UI does not decide what data to display.** That decision belongs to the product plugin. The UI only displays what it receives.

**The UI is not responsible for infrastructure.** Storage, routing, and other base services do not belong to the UI layer.

**The UI does not know product rules.** The UI must not know, for example, that an account balance cannot go negative. That rule lives in the product plugin.

## Data Flow

Data in Bonyan flows in two directions:

**From plugin to UI:** The plugin prepares data and makes it available to the UI. The UI displays it.

**From UI to plugin:** The user performs an action or enters data. The UI publishes the appropriate event. The plugin listens, processes the data, and if needed makes updated data available to the UI.

The UI never directly modifies the state of a plugin.

## The Display and Logic Boundary

There is a clear line that must be respected:

**Allowed:** Display formatting such as localizing a date, showing a number with thousand separators, or changing color based on a value. Also allowed is visual sorting or filtering that only changes how data is presented, not the underlying data itself.

**Not allowed:** Real data filtering, calculations, validation of product rules, or any decision that relates to business logic.

## Implementation Recommendation

Bonyan does not impose any specific technology for the UI layer. The choice of technology is the developer's.

That said, Web Components are recommended. Web Components are a browser standard, depend on no framework, and work without modification in Capacitor and Tauri. This aligns with Bonyan's principles of being framework-free and cross-platform.