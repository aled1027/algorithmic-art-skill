# AGENTS.md - Algorithmic Art Skill

## Commands
- **Development**: `npm run dev` (starts Vite dev server at localhost:5173)
- **Build**: `npm run build` (production build)
- **Check types**: `npm run check` (svelte-check with TypeScript)
- **Format**: `npm run format` (Prettier on src/**/*.{js,ts,svelte})
- **Preview**: `npm run preview` (test production build locally)

## Architecture
- **SvelteKit 5** framework with TypeScript, Vite bundler
- **p5.js** for algorithmic art/sketches (imported and initialized in +page.svelte)
- **Tailwind CSS 4** for styling (@tailwindcss/vite integration)
- Routes in `src/routes/` (file-based routing), shared code in `src/lib/`
- Single-page app with canvas container; p5 lifecycle tied to component onMount/unmount

## Code Style
- **TypeScript**: strict mode enabled (`strict: true`), use explicit types
- **Imports**: Use `$lib` alias for `src/lib/` imports; ES modules only
- **Components**: Svelte 5 syntax (runes), use `bind:` for reactive refs, `onMount` for setup
- **Naming**: camelCase for variables/functions, PascalCase for components/types
- **Formatting**: spaces (no tabs), single quotes, no trailing commas, 100 char width
- **p5 lifecycle**: Always cleanup in onMount return (call `p5Instance.remove()`)
