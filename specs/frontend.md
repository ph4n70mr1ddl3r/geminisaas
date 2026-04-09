# Frontend Specification

## Technology Stack

- **Framework:** [Leptos](https://github.com/leptos-rs/leptos) (Full-stack SSR/CSR) or [Dioxus](https://github.com/DioxusLabs/dioxus).
- **Styling:** Vanilla CSS or CSS-in-JS (via `leptos_meta` and scoped styles). **Avoid TailwindCSS** unless specified otherwise.
- **State Management:** Leptos signals or Dioxus hooks.
- **Routing:** Built-in router for the chosen framework.
- **Serialization:** `serde-wasm-bindgen` for efficient WASM-JS interop.

## UI/UX Principles

1.  **Density:** ERP UIs are data-heavy. Prioritize data density (compact tables, multi-column forms).
2.  **Navigation:** Persistent side navigation for modules, top bar for global search and user profile.
3.  **Feedback:** Immediate visual feedback for all actions (loading states, success/error toast notifications).
4.  **Consistency:** Use a shared component library for buttons, inputs, tables, and modals.
5.  **Accessibility:** Follow WCAG 2.1 standards for all UI components.

## Interaction Patterns

- **Server-Side Rendering (SSR):** Preferred for initial page load and SEO (if applicable).
- **Hydration:** Client-side interactivity via WASM.
- **Optimistic UI:** Update local state before server confirmation for better UX.
- **Pagination & Sorting:** Server-side pagination for all tables to handle large datasets.
- **Reporting:** Export to CSV/PDF should be handled by the backend.

## Design System

- **Colors:** Professional blue/gray palette similar to Oracle's "Alta" or "Redwood" design systems.
- **Typography:** Sans-serif (e.g., Inter, Roboto).
- **Icons:** SVG-based icons (e.g., Lucide or FontAwesome).
