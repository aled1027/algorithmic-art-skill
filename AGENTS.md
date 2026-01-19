# AGENTS.md - Algorithmic Art Skill

## Build/Test/Lint Commands
- `npm run dev` - Start dev server on localhost:5173
- `npm run build` - Build production bundle
- `npm run check` - Type-check and lint with svelte-check
- `npm run check:watch` - Watch mode for type-checking
- `npm run preview` - Preview production build locally

## Project Structure
**SvelteKit project** - Modern Svelte 5 + TypeScript + Vite

- `src/routes/` - SvelteKit file-based routing (+page.svelte, +layout.svelte)
- `src/lib/` - Reusable components and utilities (imported via `$lib` alias)
- `src/lib/assets/` - Static assets
- `src/app.d.ts` - Global TypeScript types
- `static/` - Public static files
- `vite.config.ts` - Vite configuration
- `svelte.config.js` - SvelteKit configuration

## Code Style & Conventions
- **TypeScript**: Strict mode enabled, ESM modules, no .js extensions in imports
- **Imports**: Use `$lib` alias for local imports (e.g., `import Foo from '$lib/components/Foo.svelte'`)
- **Formatting**: SvelteKit standard; use svelte-check for validation
- **Components**: Svelte 5 syntax (.svelte files in routes and lib)
- **Error Handling**: Use TypeScript for compile-time safety; SvelteKit error pages in routes
- **Naming**: kebab-case for files, PascalCase for components, camelCase for functions/variables
