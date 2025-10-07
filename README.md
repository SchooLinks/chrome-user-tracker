# SchooLinks User Tracker — Chrome Extension

SchooLinks User Tracker is a Chrome extension that enhances the SchooLinks Django Admin experience. It adds a “Favorite” action next to user/district rows in Admin lists and provides a popup UI to quickly find, launch, and manage your favorites across multiple environments.

- Save favorites directly from Django Admin (per environment)
- Quick-launch Instant (and Local in dev mode) login links
- Search by name, email, district, id, user type, grade, or notes
- Add/remove notes to any favorite
- Environment selector (dev, qa, staging, prod, etc.) with color-coding
- Optional VPN requirement check for Admin access
- Import/Export full backup of your saved data

For a non-technical overview and usage context, see: https://www.notion.so/schoolinkshome/SchooLinks-User-Tracker-Extension-11c7e901e51448158e34d183822611a8?utm_content=11c7e901-e514-4815-8e34-d183822611a8&utm_campaign=T06PNNACA&n=slack&n=slack_link_unfurl&pvs=6

## How it works

- Content script injection on Django Admin list pages (except user edit pages) adds a "Favorite" link to each row. Clicking it captures key fields from the row and stores them locally in browser storage, namespaced by environment.
- The popup UI lets you choose an environment, search favorites, launch Instant/Local links, add/remove notes, and remove favorites. It also supports backing up and restoring data as JSON.

## Project structure

- manifest.json — Chrome MV3 manifest and content script registration (matches https://*.schoolinks.com/sl-admin/* with one exclude path)
- content-script/
  - src/main.tsx — Injects a React root into the page and renders the content script App
  - src/App.tsx — Parses Admin table rows, adds a "Favorite" link, stores selected row data in storage
- src/ (Popup)
  - main.tsx — Renders the popup App
  - App.tsx — Environment selector, VPN check, search, notes, quick links, import/export
  - Row.tsx — Per-favorite UI (expand, notes, remove, open links)
  - ConfirmationPopover.tsx — Small confirmation component used by Row
- lib/
  - utils.ts — Utility helpers and wrapper around chrome.storage.local (getSyncData, setSyncData), and getFullName
  - constants.ts — List of supported environments (dev-*, qa, staging, stable, prod, etc.)
- components/ — Reusable UI primitives (buttons, inputs, dialogs, popovers, toggles, etc.)

## Key behaviors and data model

- Environments
  - Defined in lib/constants.ts. The popup presents these and persists the last selected environment under key selectedEnv.
  - Dev mode (toggle in the popup config) reveals additional dev environments and shows Local links when not in prod.
- Storage
  - This extension uses chrome.storage.local. Data is organized as { [env]: { [id]: rowData } }.
  - Import/Export in the popup reads/writes the entire storage JSON for easy backups.
- VPN check
  - On popup open, the extension requests https://app.schoolinks.com/sl-admin/login/?next=/sl-admin/. A 403 response indicates VPN is required and shows an advisory.
  - Typing opensesame123 in the search box bypasses the VPN advisory for emergency access during testing.
- Links
  - Instant and Local links are surfaced from captured row data. For district admin links, a query parameter is toggled to generate Local vs Instant variants.

## Setup

1. Install dependencies
   - yarn
2. Build the extension
   - yarn build
   - Output is written to dist/
3. Load in Chrome
   - Open chrome://extensions/
   - Enable Developer mode
   - Click Load unpacked and select the dist directory
4. Optional: Development workflow
   - yarn dev starts Vite. When developing the popup UI, you can iterate quickly.
   - For content-script changes, rebuild and reload the extension to see updates on Admin pages.

## Using the extension

1. Visit SchooLinks Django Admin lists that include login links
2. Click Favorite on any row you want to keep handy
3. Open the extension popup
   - Pick the environment (top-left button)
   - Use search to filter favorites by multiple fields or notes
   - Click Instant (and Local in dev mode, non‑prod) to open login links
   - Add comma-separated notes, press Enter to save
   - Click the trash icon to remove a favorite
4. Import/Export backups from the Config dialog (info icon)
5. Toggle Dev Mode to reveal more environments and Local links

## Permissions

- storage — Required to save your favorites and settings locally

## Privacy and data

- All data is stored locally in chrome.storage.local on your machine
- No data is transmitted to external services by this extension

## Troubleshooting

- I only see “Unable to access” in the popup
  - Ensure your VPN is connected if required to reach the Admin site.
- I don’t see Local links
  - Enable Dev Mode in the popup configuration and ensure the current environment is not prod.
- Favorites disappeared after switching environment
  - Favorites are stored per environment. Switch back to the prior environment.

## Scripts

- yarn dev — Start Vite for development
- yarn build — Type-check and build the extension (dist/)
- yarn preview — Preview the built popup

## License

Internal/Private — Please consult your team before distributing.
