# Design System Specification: The Executive Architect

## 1. Overview & Creative North Star
**Creative North Star: "The Digital Curator"**
This design system moves away from the "industrial" feel of traditional HRIS platforms toward an editorial, high-end workspace. The goal is to balance the gravity of human resources—trust, legality, and career growth—with the fluid efficiency of a modern SaaS.

We reject the "boxed-in" layout. Instead of a grid of rigid containers, we use **Intentional Asymmetry** and **Tonal Depth**. By overlapping elements and using a sophisticated scale of surface colors, we create a UI that feels like a series of layered, premium documents on a clean desk. This approach ensures that even data-dense screens feel breathable and authoritative.

---

## 2. Colors & Surface Philosophy
The palette is rooted in `primary` (#091426) and `secondary` (#0058be), creating a foundation of stability.

*   **The "No-Line" Rule:** 1px solid borders are strictly prohibited for sectioning. Use background shifts (e.g., a `surface-container-low` panel sitting on a `surface` background) to define boundaries.
*   **Surface Hierarchy & Nesting:** Use the `surface-container` tiers to create depth. 
    *   **Base:** `surface` (#f7f9fb)
    *   **Low-Level Sections:** `surface-container-low` (#f2f4f6)
    *   **Interactive Cards:** `surface-container-lowest` (#ffffff)
*   **The "Glass & Gradient" Rule:** For floating navigation or high-level summaries, use Glassmorphism. Set the background to `surface_container_lowest` at 80% opacity with a 12px backdrop-blur. 
*   **Signature Textures:** Main CTAs should not be flat. Apply a subtle linear gradient from `primary` (#091426) to `primary_container` (#1e293b) at a 135-degree angle to provide "visual soul."

---

## 3. Typography
We use a dual-font strategy to separate "Data" from "Direction."

*   **Display & Headlines (Manrope):** The wider aperture and geometric builds of Manrope convey a modern, architectural feel. Use `display-md` for dashboard greetings and `headline-sm` for section titles to establish a strong editorial voice.
*   **Body & Labels (Inter):** Inter is our workhorse. It provides maximum legibility for dense employee records and payroll data. 
*   **Hierarchy Note:** Use `on_surface_variant` (#45474c) for secondary metadata to create a natural "gray-scale" hierarchy that guides the eye without needing bold weights.

---

## 4. Elevation & Depth
Depth is achieved through **Tonal Layering**, not structural lines.

*   **The Layering Principle:** To lift a profile card, place a `surface-container-lowest` card atop a `surface-container` background. This "soft lift" is more professional than heavy shadows.
*   **Ambient Shadows:** When an element must float (e.g., a dropdown or modal), use an ultra-diffused shadow: `0 20px 50px -12px rgba(25, 28, 30, 0.08)`. The shadow color is a tint of `on_surface`, never pure black.
*   **The "Ghost Border" Fallback:** If a divider is required for accessibility, use `outline_variant` (#c5c6cd) at **15% opacity**. This creates a "suggestion" of a line rather than a hard break.

---

## 5. Components & Interaction

### Sidebars & Navigation
*   **Style:** Use `primary_container` (#1e293b) for the sidebar background.
*   **Active State:** Do not use a box. Use a vertical "pill" indicator (4px wide) in `secondary` (#0058be) on the leading edge, with the text shifting to `on_secondary_container`.

### Data Tables (The "Density" Hero)
*   **Header:** `label-md` in uppercase with 0.05em tracking. Use `surface_container_high` for the header background.
*   **Rows:** Forbid horizontal dividers. Use a `2.5` (0.85rem) vertical padding. On hover, transition the row background to `surface_container_low`.
*   **Status Badges:** Use "Soft Fills." 
    *   *Success:* `surface_container` with `on_secondary_fixed_variant` text. 
    *   *Error:* `error_container` with `on_error_container` text.
    *   *Shape:* Use `full` roundedness for a pill-style look.

### Interactive Form Elements
*   **Inputs:** Use `surface_container_lowest` for the field fill. Instead of a 1px border, use a 2px bottom-stroke of `outline_variant` that transforms into `secondary` (#0058be) on focus.
*   **Buttons:**
    *   *Primary:* Gradient fill (Primary to Primary Container), `xl` (0.75rem) roundedness.
    *   *Tertiary:* No container. `label-md` bold text with a `3` (1rem) icon-gap.

### Advanced HR Components
*   **The "Employee Spotlight" Card:** An asymmetrical layout where the profile photo overlaps the top-left edge of a `surface_container_lowest` card by `4` (1.4rem).
*   **Timeline Feeds:** Use a "Threaded" look. No lines—only vertical spacing of `6` (2rem) and `surface_dim` small dots to indicate chronological nodes.

---

## 6. Do's and Don'ts

### Do
*   **Do** use white space as a structural element. If a screen feels cluttered, increase the spacing scale from `4` to `6` before adding a border.
*   **Do** use `secondary` (#0058be) sparingly for "Actionable Intel"—buttons, links, or status updates.
*   **Do** ensure text contrast ratios meet AA standards by utilizing the `on_surface` and `on_primary` tokens strictly.

### Don't
*   **Don't** use 100% black (#000000). Always use `on_surface` (#191c1e) for text to maintain a premium, ink-like feel.
*   **Don't** use standard "Drop Shadows." If an element isn't physically "floating" (like a menu), it shouldn't have a shadow. Use Tonal Layering instead.
*   **Don't** crowd data. If a table has 12 columns, use a horizontal scroll on a `surface` layer rather than shrinking the typography below `body-sm`.