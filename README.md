# Dreamz

Dreamz is a playful, pixel-inspired dream journal built with SvelteKit. The app presents a vertically stacked timeline of dream entries and a focused creation screen for logging new memories with optional mood tags. All state is handled client-side, making it a lightweight base for experimenting with richer storage, syncing, or AI-assisted storytelling features.

## Features

- **Dream timeline** – Groups entries by day with human-friendly labels so you can quickly skim past nights.
- **Quick add flow** – A floating action button opens a full-screen form with autosizing fields for the title and dream content.
- **Mood tracking** – Toggleable happy/sad buttons let you capture emotional context for each dream.
- **Keyboard and pointer friendly** – Custom pixel-press interactions and focus styles keep the UI accessible.
- **Custom assets** – Bundles bespoke pixel art icons and the Handjet typeface to reinforce the retro vibe.

## Project structure

```
├── package.json         # npm scripts and dev dependencies
├── src
│   ├── routes
│   │   ├── +layout.svelte   # Root layout wrapper
│   │   └── +page.svelte     # Main dream timeline & form logic
│   ├── design_items         # Icons, fonts, and other visual assets
│   └── lib                  # (Reserved) shared utilities/components
├── static                  # Files copied verbatim to the build output
├── svelte.config.js        # SvelteKit configuration
└── vite.config.ts          # Vite bundler configuration
```

## Getting started

### Prerequisites

- Node.js 18 or newer (matches the SvelteKit/Vite support policy)
- npm (bundled with Node) or another compatible package manager

### Installation

```bash
npm install
```

### Development server

Start the live-reloading dev server on <http://localhost:5173>:

```bash
npm run dev
```

Add `-- --open` to automatically launch your browser after the server boots.

### Type checking

Use Svelte Check to run TypeScript and component diagnostics:

```bash
npm run check
```

### Production build

Create an optimized production bundle:

```bash
npm run build
```

Preview the built app locally (simulates deployment output):

```bash
npm run preview
```

## Extending the app

- **Persist data** – Connect the creation flow to a database, Supabase, or local storage to retain dreams between sessions.
- **Categorize further** – Swap the static `categories` array for a filterable store fed by real data.
- **Enrich entries** – Add fields for tags, people, or lucid dreaming notes, or integrate AI summarization.

## License

This project inherits the default SvelteKit template license (MIT) included in `LICENSE`. Custom assets in `src/design_items` retain their original terms—see the bundled font README for Handjet attribution details.
