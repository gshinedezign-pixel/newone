This repository is a Shopify theme (Liquid + static assets). The guidance below is short, actionable, and focused on patterns an AI coding agent needs to be immediately productive.

Big picture
- **Platform:** Shopify theme (Liquid templates + `assets/` JS/CSS). Key top-level folders: `assets/`, `sections/`, `snippets/`, `templates/`, `locales/`, `config/`, `layout/`.
- **Runtime model:** Server-rendered Liquid templates with progressive enhancement via modular JS in `assets/` (no bundler pipeline present in repo). JavaScript files map to UI components and are loaded as standalone assets.

Key architectural patterns (concrete examples)
- **Custom elements:** UI components are implemented as custom elements. Example: `assets/product-form.js` defines a `product-form` custom element; `assets/global.js` defines `quantity-input` custom element.
- **Pub/sub event bus:** Cross-component communication uses `assets/pubsub.js`'s `subscribe(event, cb)` (returns an unsubscribe function) and `publish(event, data)` (returns a Promise). See `product-form.js` publishing `PUB_SUB_EVENTS.cartUpdate` and other components subscribing.
- **DOM update utility:** `assets/global.js` provides `HTMLUpdateUtility.viewTransition(oldNode, newContent, ...)` and `setInnerHTML(...)` to replace sections and safely re-execute scripts — prefer these helpers when swapping server-rendered HTML fragments.
- **Section ID convention:** `assets/global.js` contains `SectionId` helper which uses `__` as a separator in qualified section IDs (e.g. `template--123__main`). Use `SectionId.parseId()` / `parseSectionName()` when manipulating section identifiers.
- **AJAX section rendering flow:** Components append `sections` and `sections_url` to form data (see `product-form.js`) and call Shopify endpoints (e.g. `routes.cart_add_url`) — cart/UI updates expect returned HTML for sections and use `.renderContents(response)` on cart components (`cart-drawer`, `cart-notification`). Follow that flow when adding remote-updated UI.
- **Helper utilities available globally:** `fetchConfig(type)`, `CartPerformance` markers, `routes` and localization strings such as `window.variantStrings` are used across assets — inspect `assets/constants.js` and `locales/` for names/keys.

Developer workflows (what agents should run)
- **Pushing changes to preview/store:** this repo uses the Shopify CLI. Example observed in terminal: `shopify theme push --store <my-store>.myshopify.com`. Use that to upload theme changes to a store.
- **Version control:** edits are committed to Git as usual (`git push` is used in this workspace).
- **Local edit cycle:** Edit Liquid or files in `assets/` directly; test by pushing to a preview store or using the Shopify CLI dev commands if available in environment.

Project-specific conventions and checks
- **No JS bundler:** JavaScript lives in `assets/` as individual files. Do not introduce build-step artifacts unless adding an explicit build pipeline and updating theme asset references.
- **Naming conventions:** Many components use hyphenated custom element names (`product-form`, `quantity-input`, `quick-add-modal`, `cart-drawer`). Match these names when adding markup or querying elements.
- **Event names/constants:** Event names live in `assets/constants.js` (look for `PUB_SUB_EVENTS` or similar). Use these constants rather than hard-coded strings.
- **Section updates via AJAX:** When returning server-rendered HTML for partial updates, prefer the existing pattern: return section HTML and let cart/section components call `HTMLUpdateUtility.setInnerHTML` / `viewTransition` or the cart's `.renderContents(response)` method.

Files to inspect when making changes (high value)
- `assets/global.js` — DOM utilities, SectionId, accessibility helpers.
- `assets/pubsub.js` — subscribe/publish API and expected behavior (unsubscribe returned, publish returns Promise).
- `assets/product-form.js` — canonical example of AJAX add-to-cart flow and use of `fetchConfig`, `routes`, `publish()`.
- `assets/cart-drawer.js`, `assets/cart-notification.js` — cart rendering and `getSectionsToRender()` / `renderContents()` patterns.
- `assets/constants.js` — event names, route keys, and other shared constants.
- `sections/` and `snippets/` — Liquid components you will update when changing templates.

What to avoid doing automatically
- Do not add a build step that compiles to different filenames without updating references in `layout/` or `theme.liquid`.
- Do not assume a global state shape beyond what's provided by `routes`, `window.variantStrings`, and constants in `assets/constants.js`.

Examples (copy/paste patterns)
- Subscribe/unsubscribe:
  - `const unsub = subscribe(PUB_SUB_EVENTS.cartUpdate, data => { /* .. */ });` then `unsub();`
- Publish and wait for listeners:
  - `publish(PUB_SUB_EVENTS.cartUpdate, { source: 'product-form', cartData: response }).then(() => { /* after listeners */ });`
- Push theme to store (example):
  - `shopify theme push --store gulshan23.myshopify.com`

If anything in this file is unclear or you want more detail about a specific area (AJAX flows, a particular section, or how localization is used), tell me which files or flows you'd like expanded and I will update this guidance.
