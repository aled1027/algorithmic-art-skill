# AGENTS.md - Algorithmic Art Skill

## Commands
- `npm run dev` - Vite dev server at localhost:5173
- `npm run build` - Production build
- `npm run check` - TypeScript/Svelte type checking (no test runner)
- `npm run format` - Prettier formatting

## Architecture
- **SvelteKit 5** + TypeScript + Vite + Tailwind CSS 4
- **p5.js** for canvas-based algorithmic art
- `src/routes/+page.svelte` - Main page with animation switcher
- `src/lib/animations/*.svelte` - Individual art components (p5-based)
- Each animation is a self-contained Svelte component with p5 lifecycle

## Code Style
- TypeScript strict mode; explicit types required
- Use `$lib` alias for imports from `src/lib/`
- Svelte 5 runes syntax; `onMount` for p5 setup, cleanup via return callback
- camelCase for variables/functions, PascalCase for components
- Formatting: spaces, single quotes, no trailing commas, 100 char width
- p5 cleanup: Always call `p5Instance.remove()` in onMount return
