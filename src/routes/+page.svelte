<script lang="ts">
  import FlowFieldLandscape from '$lib/animations/FlowFieldLandscape.svelte';
  import GeometricTessellation from '$lib/animations/GeometricTessellation.svelte';
  import CellularAutomata from '$lib/animations/CellularAutomata.svelte';
  import RecursiveTree from '$lib/animations/RecursiveTree.svelte';
  import ParticleGalaxy from '$lib/animations/ParticleGalaxy.svelte';
  import AudioWaveforms from '$lib/animations/AudioWaveforms.svelte';
  import GenerativeTopography from '$lib/animations/GenerativeTopography.svelte';
  import VoronoiStainedGlass from '$lib/animations/VoronoiStainedGlass.svelte';
  import BrushStrokeSimulator from '$lib/animations/BrushStrokeSimulator.svelte';
  import FractalFlame from '$lib/animations/FractalFlame.svelte';

  const PAGE_SIZE = 1;
  let currentPage = $state(0);

  const animations = [
    {
      name: 'Flow Field Landscape',
      component: FlowFieldLandscape,
      description:
        'Particles follow Perlin noise-driven vector fields, creating organic, wind-like patterns that evoke rolling hills or ocean currents'
    },

    {
      name: 'Recursive Tree Generator',
      component: RecursiveTree,
      description:
        'L-system inspired branching structures with randomized angles, lengths, and colors—producing unique botanical forms each time'
    },
    {
      name: 'Geometric Tessellation',
      component: GeometricTessellation,
      description:
        'Interlocking shapes (triangles, hexagons, or Penrose tiles) with procedural color harmonies and subtle animations'
    },
    {
      name: 'Particle Galaxy',
      component: ParticleGalaxy,
      description:
        'Thousands of particles orbiting a central attractor with gravitational physics, forming spiral arm structures'
    },
    {
      name: 'Cellular Automata',
      component: CellularAutomata,
      description:
        "Conway's Game of Life or custom rule sets rendered with color trails, showing emergent complexity from simple rules"
    },
    {
      name: 'Audio-Reactive Waveforms',
      component: AudioWaveforms,
      description:
        'Sine waves, Lissajous curves, and interference patterns that can respond to interactive frequency parameters'
    },
    {
      name: 'Brush Stroke Simulator',
      component: BrushStrokeSimulator,
      description:
        'Simulated paint strokes with varying pressure, opacity, and texture—building up abstract expressionist compositions'
    },
    {
      name: 'Generative Topography',
      component: GenerativeTopography,
      description:
        'Layered contour lines derived from noise functions, creating abstract mountain range maps with elevation coloring'
    },
    {
      name: 'Fractal Flame Renderer',
      component: FractalFlame,
      description:
        'Iterated function systems generating the intricate, luminous structures seen in classic fractal flame art'
    },
    {
      name: 'Voronoi Stained Glass',
      component: VoronoiStainedGlass,
      description:
        'Dynamically generated Voronoi diagrams with gradient fills and dark edges, mimicking cathedral windows'
    }
  ];

  const totalPages = Math.ceil(animations.length / PAGE_SIZE);
  const paginatedAnimations = $derived(
    animations.slice(currentPage * PAGE_SIZE, (currentPage + 1) * PAGE_SIZE)
  );

  function prevPage() {
    if (currentPage > 0) currentPage--;
  }

  function nextPage() {
    if (currentPage < totalPages - 1) currentPage++;
  }
</script>

<main class="min-h-screen bg-gray-50 px-6 py-12 flex justify-center">
  <div class="w-full max-w-[400px]">
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

    <div class="mt-8">
      {#each paginatedAnimations as animation (animation.name)}
        <div>
          <h2 class="mb-2 text-xl font-semibold text-gray-800">
            {animation.name}
          </h2>
          <p class="mb-4 text-sm text-gray-600">
            {animation.description}
          </p>
          <div class="animation-wrapper">
            <animation.component />
          </div>
        </div>
      {/each}

      <div class="mt-8 flex items-center justify-center gap-4">
        <button
          onclick={prevPage}
          disabled={currentPage === 0}
          class="rounded bg-gray-200 px-4 py-2 font-medium text-gray-700 hover:bg-gray-300 disabled:cursor-not-allowed disabled:opacity-50"
        >
          Previous
        </button>
        <span class="text-gray-600">
          Page {currentPage + 1} of {totalPages}
        </span>
        <button
          onclick={nextPage}
          disabled={currentPage === totalPages - 1}
          class="rounded bg-gray-200 px-4 py-2 font-medium text-gray-700 hover:bg-gray-300 disabled:cursor-not-allowed disabled:opacity-50"
        >
          Next
        </button>
      </div>
    </div>
  </div>
</main>

<style>
  .animation-wrapper {
    position: relative;
    width: 400px;
    height: 400px;
    border-radius: 10px;
    overflow: hidden;
  }
</style>
