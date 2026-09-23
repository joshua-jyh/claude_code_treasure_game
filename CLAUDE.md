# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install
npm run dev     # Vite dev server on http://localhost:3000 (auto-opens browser)
npm run build   # Production build to ./build (not ./dist)
```

There is no lint, typecheck, or test setup (no ESLint, no `tsconfig.json`, no test runner). Vite with `@vitejs/plugin-react-swc` transpiles TypeScript without type-checking, so type errors will not surface at build time.

## Architecture

A single-page React 18 + Vite treasure-hunt game, originally exported from Figma Make (hence `src/components/figma/` and `src/guidelines/Guidelines.md`, which is an unfilled template).

- **All game logic and UI live in `src/App.tsx`.** State is three `useState` hooks: `boxes` (3 chests, one randomly holding treasure), `score` (+100 for treasure, −50 for skeleton), and `gameEnded` (true once the treasure is found or all chests are opened). `initializeGame()` runs on mount and on "Play Again".
- Animations use `motion/react` (Framer Motion's successor package).
- Static assets (chest images in `src/assets/`, sound effects in `src/audios/`) are imported as ES modules and resolved by Vite. `src/results/` holds reference screenshots, not runtime assets.
- `src/components/ui/` is a vendored shadcn/ui component set (Radix primitives + `cva` + `cn()` from `ui/utils.ts`). Only `Button` is currently used.

## Gotchas

- **Tailwind is not installed.** `src/index.css` is a *pre-compiled* Tailwind v4 output stylesheet imported by `main.tsx`. Only utility classes that already exist in that file will work; new classes (e.g. an arbitrary `cursor-[url(...)]`) will silently have no effect. Use inline `style` props or add hand-written CSS rules to `index.css` instead. `src/styles/globals.css` (theme tokens) is not imported anywhere.
- **Versioned import specifiers.** shadcn components import packages like `"@radix-ui/react-slot@1.1.2"`. These resolve only because `vite.config.ts` aliases each `pkg@version` to `pkg`. When adding a dependency imported this way, add a matching alias (or import the bare package name).
- `@` is aliased to `./src`.
- Asset credits are tracked in `Attributions.md`; add an entry when introducing new third-party images or sounds.
