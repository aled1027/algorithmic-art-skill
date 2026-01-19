<script lang="ts">
  import { onMount } from 'svelte';
  import p5 from 'p5';

  let container: HTMLDivElement;
  let p5Instance: p5 | null = null;

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        const circles: { x: number; y: number; size: number; color: p5.Color }[] = [];

        sketch.setup = () => {
          sketch.createCanvas(container.clientWidth, container.clientHeight);
        };

        sketch.draw = () => {
          sketch.background(30);
          sketch.noStroke();
          for (const circle of circles) {
            sketch.fill(circle.color);
            sketch.ellipse(circle.x, circle.y, circle.size);
          }
        };

        sketch.mousePressed = () => {
          if (
            sketch.mouseX >= 0 &&
            sketch.mouseX <= sketch.width &&
            sketch.mouseY >= 0 &&
            sketch.mouseY <= sketch.height
          ) {
            circles.push({
              x: sketch.mouseX,
              y: sketch.mouseY,
              size: sketch.random(20, 60),
              color: sketch.color(
                sketch.random(100, 255),
                sketch.random(100, 255),
                sketch.random(100, 255),
                200
              )
            });
          }
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
