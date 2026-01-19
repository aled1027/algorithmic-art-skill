<script lang="ts">
  import OrbitingCircle from '$lib/animations/OrbitingCircle.svelte';
  import ClickCircles from '$lib/animations/ClickCircles.svelte';
  import FlowFieldLandscape from '$lib/animations/FlowFieldLandscape.svelte';
  import GeometricTessellation from '$lib/animations/GeometricTessellation.svelte';
  import CellularAutomata from '$lib/animations/CellularAutomata.svelte';
  import RecursiveTree from '$lib/animations/RecursiveTree.svelte';
  import ParticleGalaxy from '$lib/animations/ParticleGalaxy.svelte';
  import AudioWaveforms from '$lib/animations/AudioWaveforms.svelte';
  import GenerativeTopography from '$lib/animations/GenerativeTopography.svelte';
  import VoronoiStainedGlass from '$lib/animations/VoronoiStainedGlass.svelte';

  const animations = [
    { name: 'Flow Field Landscape', component: FlowFieldLandscape },
    { name: 'Voronoi Stained Glass', component: VoronoiStainedGlass },
    { name: 'Recursive Tree Generator', component: RecursiveTree },
    { name: 'Geometric Tessellation', component: GeometricTessellation },
    { name: 'Particle Galaxy', component: ParticleGalaxy },
    { name: 'Cellular Automata', component: CellularAutomata },
    { name: 'Audio-Reactive Waveforms', component: AudioWaveforms },
    { name: 'Generative Topography', component: GenerativeTopography },
    { name: 'Orbiting Circle', component: OrbitingCircle },
    { name: 'Click Circles', component: ClickCircles }
  ];
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

    <div class="mt-8 grid grid-cols-1 md:grid-cols-2 gap-8">
      {#each animations as animation}
        <div>
          <h2 class="mb-4 text-xl font-semibold text-gray-800">
            {animation.name}
          </h2>
          <div class="animation-wrapper">
            <animation.component />
          </div>
        </div>
      {/each}
    </div>
  </div>
</main>

<style>
  .animation-wrapper {
    position: relative;
    width: 350px;
    height: 350px;
    border-radius: 10px;
    overflow: hidden;
  }
</style>
