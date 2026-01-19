# Animation Gallery Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a gallery system for p5.js animation components on the main page.

**Architecture:** Extract animations into standalone Svelte components in `src/lib/animations/`. Each component manages its own p5 instance. The main page imports and renders all animations in a grid gallery.

**Tech Stack:** SvelteKit 5, p5.js, TypeScript, Tailwind CSS

---

### Task 1: Create Animation Component Directory Structure

**Files:**
- Create: `src/lib/animations/OrbitingCircle.svelte`

**Step 1: Create the animations directory and OrbitingCircle component**

Create `src/lib/animations/OrbitingCircle.svelte` with the existing p5 animation logic extracted:

```svelte
<script lang="ts">
  import { onMount } from "svelte"
  import p5 from "p5"

  let container: HTMLDivElement
  let p5Instance: p5 | null = null

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        sketch.setup = () => {
          sketch.createCanvas(400, 300)
        }

        sketch.draw = () => {
          sketch.background(220)
          sketch.fill(100, 150, 255)
          sketch.noStroke()
          sketch.ellipse(
            sketch.width / 2 + sketch.sin(sketch.frameCount * 0.05) * 100,
            sketch.height / 2 + sketch.cos(sketch.frameCount * 0.05) * 100,
            50,
            50,
          )
        }
      }, container)
    }

    return () => {
      if (p5Instance) {
        p5Instance.remove()
      }
    }
  })
</script>

<div bind:this={container} class="overflow-hidden rounded-lg shadow-lg"></div>
```

**Step 2: Verify the file was created**

Run: `ls -la src/lib/animations/`
Expected: OrbitingCircle.svelte exists

**Step 3: Commit**

```bash
git add src/lib/animations/OrbitingCircle.svelte
git commit -m "feat: extract OrbitingCircle animation component"
```

---

### Task 2: Update +page.svelte to Gallery Layout

**Files:**
- Modify: `src/routes/+page.svelte`

**Step 1: Replace +page.svelte with gallery structure**

Update `src/routes/+page.svelte`:

```svelte
<script lang="ts">
  import OrbitingCircle from "$lib/animations/OrbitingCircle.svelte"

  const animations = [
    { name: "Orbiting Circle", component: OrbitingCircle }
  ]
</script>

<main class="min-h-screen bg-gray-50 px-6 py-12">
  <div class="mx-auto max-w-5xl">
    <h1 class="text-3xl font-bold text-gray-900">Algorithmic Art Gallery</h1>
    <p class="mt-2 text-gray-600">
      A gallery of p5.js animations created with the
      <a
        href="https://claude-plugins.dev/skills/@smallnest/goskills/algorithmic-art"
        class="text-blue-600 underline hover:text-blue-800"
      >
        algorithmic art skill</a
      >.
    </p>

    <div class="mt-8 grid gap-8">
      {#each animations as animation}
        <div class="rounded-xl bg-white p-4 shadow">
          <h2 class="mb-4 text-xl font-semibold text-gray-800">{animation.name}</h2>
          <animation.component />
        </div>
      {/each}
    </div>
  </div>
</main>
```

**Step 2: Type check the changes**

Run: `npm run check`
Expected: No errors

**Step 3: Start dev server and verify visually**

Run: `npm run dev`
Expected: Gallery displays with "Orbiting Circle" animation visible

**Step 4: Commit**

```bash
git add src/routes/+page.svelte
git commit -m "feat: convert page to animation gallery layout"
```

---

## Adding New Animations

To add a new animation to the gallery:

1. Create a new component in `src/lib/animations/YourAnimation.svelte`
2. Import it in `+page.svelte`
3. Add an entry to the `animations` array: `{ name: "Your Animation", component: YourAnimation }`
