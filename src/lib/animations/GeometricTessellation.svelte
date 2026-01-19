<script lang="ts">
  import { onMount } from 'svelte';
  import p5 from 'p5';

  let container: HTMLDivElement;
  let p5Instance: p5 | null = null;

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        const seed = 7891;
        const baseHue = 200;
        const recursionDepth = 4;
        const lineWeight = 1.5;
        const jitterAmount = 0.02;

        interface Triangle {
          x1: number;
          y1: number;
          x2: number;
          y2: number;
          x3: number;
          y3: number;
          depth: number;
        }

        let triangles: Triangle[] = [];
        let animationProgress = 0;

        function subdivideTriangle(t: Triangle): Triangle[] {
          const mx1 = (t.x1 + t.x2) / 2;
          const my1 = (t.y1 + t.y2) / 2;
          const mx2 = (t.x2 + t.x3) / 2;
          const my2 = (t.y2 + t.y3) / 2;
          const mx3 = (t.x3 + t.x1) / 2;
          const my3 = (t.y3 + t.y1) / 2;

          const jitter = (v: number) => v + sketch.random(-jitterAmount, jitterAmount) * 20;

          return [
            { x1: t.x1, y1: t.y1, x2: jitter(mx1), y2: jitter(my1), x3: jitter(mx3), y3: jitter(my3), depth: t.depth + 1 },
            { x1: jitter(mx1), y1: jitter(my1), x2: t.x2, y2: t.y2, x3: jitter(mx2), y3: jitter(my2), depth: t.depth + 1 },
            { x1: jitter(mx3), y1: jitter(my3), x2: jitter(mx2), y2: jitter(my2), x3: t.x3, y3: t.y3, depth: t.depth + 1 },
            { x1: jitter(mx1), y1: jitter(my1), x2: jitter(mx2), y2: jitter(my2), x3: jitter(mx3), y3: jitter(my3), depth: t.depth + 1 }
          ];
        }

        function generateTessellation() {
          triangles = [];
          const w = sketch.width;
          const h = sketch.height;
          const gridSize = 3;
          const cellW = w / gridSize;
          const cellH = h / gridSize;

          for (let row = 0; row < gridSize; row++) {
            for (let col = 0; col < gridSize; col++) {
              const x = col * cellW;
              const y = row * cellH;
              triangles.push({ x1: x, y1: y, x2: x + cellW, y2: y, x3: x + cellW / 2, y3: y + cellH, depth: 0 });
              triangles.push({ x1: x, y1: y, x2: x + cellW / 2, y2: y + cellH, x3: x, y3: y + cellH, depth: 0 });
              triangles.push({ x1: x + cellW, y1: y, x2: x + cellW, y2: y + cellH, x3: x + cellW / 2, y3: y + cellH, depth: 0 });
            }
          }

          for (let d = 0; d < recursionDepth; d++) {
            const newTriangles: Triangle[] = [];
            for (const t of triangles) {
              if (sketch.random() > 0.3) {
                newTriangles.push(...subdivideTriangle(t));
              } else {
                newTriangles.push(t);
              }
            }
            triangles = newTriangles;
          }
        }

        sketch.setup = () => {
          sketch.randomSeed(seed);
          sketch.noiseSeed(seed);
          sketch.createCanvas(container.clientWidth, container.clientHeight);
          sketch.colorMode(sketch.HSB, 360, 100, 100, 100);
          generateTessellation();
        };

        sketch.draw = () => {
          sketch.background(baseHue, 5, 98);

          animationProgress = Math.min(animationProgress + 0.01, 1);
          const visibleCount = Math.floor(triangles.length * animationProgress);

          for (let i = 0; i < visibleCount; i++) {
            const t = triangles[i];
            const hue = (baseHue + t.depth * 25 + sketch.noise(t.x1 * 0.01, t.y1 * 0.01) * 40) % 360;
            const saturation = 30 + t.depth * 10;
            const brightness = 90 - t.depth * 8;

            sketch.fill(hue, saturation, brightness, 80);
            sketch.stroke(hue - 20, saturation + 20, brightness - 30);
            sketch.strokeWeight(lineWeight);
            sketch.triangle(t.x1, t.y1, t.x2, t.y2, t.x3, t.y3);
          }

          if (animationProgress >= 1) {
            sketch.noLoop();
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
