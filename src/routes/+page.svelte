<script lang="ts">
  import { onMount } from "svelte";
  import p5 from "p5";

  let container: HTMLDivElement;
  let p5Instance: p5 | null = null;

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        sketch.setup = () => {
          sketch.createCanvas(800, 600);
        };

        sketch.draw = () => {
          sketch.background(220);
          sketch.fill(100, 150, 255);
          sketch.noStroke();
          sketch.ellipse(
            sketch.width / 2 + sketch.sin(sketch.frameCount * 0.05) * 100,
            sketch.height / 2 + sketch.cos(sketch.frameCount * 0.05) * 100,
            50,
            50,
          );
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

<main class="min-h-screen bg-gray-50 px-6 py-12">
  <div class="mx-auto max-w-3xl">
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

    <div bind:this={container} class="mt-8 overflow-hidden rounded-lg shadow-lg"></div>
  </div>
</main>
