#ll  Algorithmic Art Implementation Plan

## Overview
10 generative art pieces using p5.js with seeded randomness, integrated into the SvelteKit gallery.

---

## Repository Structure

This project uses **SvelteKit 5 + p5.js + Tailwind CSS**. Each artwork is:
- A **Svelte component** in `src/lib/animations/`
- Registered in the **gallery** at `src/routes/+page.svelte`

```
src/
├── lib/
│   └── animations/
│       ├── FlowFieldLandscape.svelte      # New artwork
│       ├── RecursiveTreeGenerator.svelte  # New artwork
│       ├── ...
│       ├── ClickCircles.svelte            # Existing
│       └── OrbitingCircle.svelte          # Existing
└── routes/
    └── +page.svelte                       # Gallery (imports all animations)
```

---

## Svelte Component Pattern

Each artwork follows this p5.js-in-Svelte pattern:

```svelte
<script lang="ts">
  import { onMount } from 'svelte';
  import p5 from 'p5';

  let container: HTMLDivElement;
  let p5Instance: p5 | null = null;

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        // Seeded randomness
        const seed = 12345;
        
        sketch.setup = () => {
          sketch.randomSeed(seed);
          sketch.noiseSeed(seed);
          sketch.createCanvas(container.clientWidth, container.clientHeight);
        };

        sketch.draw = () => {
          // Algorithm here
        };
      }, container);
    }

    return () => {
      if (p5Instance) {
        p5Instance.remove();
      }
    };
  });
</script>

<div class="container" bind:this={container}></div>

<style>
  .container {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
  }
</style>
```

---

## Phase 1: Foundations

| # | Artwork | Component File | Complexity |
|---|---------|----------------|------------|
| 1 | Flow Field Landscape | `FlowFieldLandscape.svelte` | Medium |
| 6 | Geometric Tessellation | `GeometricTessellation.svelte` | Medium |
| 9 | Cellular Automata Patterns | `CellularAutomata.svelte` | Medium |

---

## Phase 2: Physics & Particles

| # | Artwork | Component File | Complexity |
|---|---------|----------------|------------|
| 4 | Particle Galaxy | `ParticleGalaxy.svelte` | High |
| 8 | Brush Stroke Simulator | `BrushStrokeSimulator.svelte` | High |

---

## Phase 3: Mathematical Beauty

| # | Artwork | Component File | Complexity |
|---|---------|----------------|------------|
| 2 | Recursive Tree Generator | `RecursiveTree.svelte` | Medium |
| 3 | Voronoi Stained Glass | `VoronoiStainedGlass.svelte` | Medium |
| 5 | Generative Topography | `GenerativeTopography.svelte` | Medium |

---

## Phase 4: Advanced Systems

| # | Artwork | Component File | Complexity |
|---|---------|----------------|------------|
| 7 | Audio-Reactive Waveforms | `AudioWaveforms.svelte` | High |
| 10 | Fractal Flame Renderer | `FractalFlame.svelte` | Very High |

---

## Per-Artwork Workflow

### Step 1: Philosophy
- Write algorithmic philosophy in `docs/{artwork-name}.md`
- 4-6 paragraphs defining computational aesthetic

### Step 2: Create Svelte Component
- Add `src/lib/animations/{ArtworkName}.svelte`
- Use p5 instance mode with `onMount`/cleanup pattern
- Implement seeded randomness with `randomSeed()`/`noiseSeed()`

### Step 3: Register in Gallery
- Import component in `src/routes/+page.svelte`
- Add to `animations` array

### Step 4: Verify
- Run `npm run dev` and check gallery at localhost:5173
- Run `npm run check` for type errors

---

## Progress Tracker

| # | Artwork | Philosophy | Component | Gallery | Status |
|---|---------|------------|-----------|---------|--------|
| 1 | Flow Field Landscape | ✅ | ✅ | ✅ | Complete |
| 2 | Recursive Tree Generator | ✅ | ✅ | ✅ | Complete |
| 3 | Voronoi Stained Glass | ✅ | ✅ | ✅ | Complete |
| 4 | Particle Galaxy | ✅ | ✅ | ✅ | Complete |
| 5 | Generative Topography | ✅ | ✅ | ✅ | Complete |
| 6 | Geometric Tessellation | ✅ | ✅ | ✅ | Complete |
| 7 | Audio-Reactive Waveforms | ✅ | ✅ | ✅ | Complete |
| 8 | Brush Stroke Simulator | ✅ | ✅ | ✅ | Complete |
| 9 | Cellular Automata Patterns | ✅ | ✅ | ✅ | Complete |
| 10 | Fractal Flame Renderer | ✅ | ✅ | ✅ | Complete |

---

## Commands

```bash
npm run dev      # Start dev server at localhost:5173
npm run check    # TypeScript/Svelte type checking
npm run build    # Production build
npm run format   # Prettier formatting
```
