<script lang="ts">
  import OrbitingCircle from '$lib/animations/OrbitingCircle.svelte';
  import ClickCircles from '$lib/animations/ClickCircles.svelte';

  const animations = [
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
