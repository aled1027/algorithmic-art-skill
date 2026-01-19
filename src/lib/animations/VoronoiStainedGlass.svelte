<script lang="ts">
  import { onMount } from 'svelte';
  import p5 from 'p5';

  let container: HTMLDivElement;
  let p5Instance: p5 | null = null;

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        const seed = 42;
        const numPoints = 40;
        const leadingWeight = 2;

        interface VoronoiPoint {
          x: number;
          y: number;
          hue: number;
        }

        let points: VoronoiPoint[] = [];
        let cellColors: Map<number, { h: number; s: number; b: number }> = new Map();

        const jewelHues = [
          0,    // Ruby red
          15,   // Amber
          120,  // Emerald green
          220,  // Sapphire blue
          280,  // Amethyst purple
          45,   // Gold
          180,  // Teal
          340   // Rose
        ];

        function findClosestPoint(x: number, y: number): number {
          let minDist = Infinity;
          let closestIdx = 0;
          for (let i = 0; i < points.length; i++) {
            const d = sketch.dist(x, y, points[i].x, points[i].y);
            if (d < minDist) {
              minDist = d;
              closestIdx = i;
            }
          }
          return closestIdx;
        }

        function isOnEdge(x: number, y: number): boolean {
          const closest = findClosestPoint(x, y);
          const closestDist = sketch.dist(x, y, points[closest].x, points[closest].y);
          
          for (let i = 0; i < points.length; i++) {
            if (i === closest) continue;
            const d = sketch.dist(x, y, points[i].x, points[i].y);
            if (Math.abs(d - closestDist) < 3) {
              return true;
            }
          }
          return false;
        }

        function countNeighbors(pointIdx: number): number {
          let neighbors = 0;
          const p = points[pointIdx];
          for (let i = 0; i < points.length; i++) {
            if (i === pointIdx) continue;
            const d = sketch.dist(p.x, p.y, points[i].x, points[i].y);
            if (d < sketch.width * 0.35) {
              neighbors++;
            }
          }
          return neighbors;
        }

        sketch.setup = () => {
          sketch.randomSeed(seed);
          sketch.noiseSeed(seed);
          sketch.createCanvas(container.clientWidth, container.clientHeight);
          sketch.colorMode(sketch.HSB, 360, 100, 100, 100);

          for (let i = 0; i < numPoints; i++) {
            const x = sketch.random(20, sketch.width - 20);
            const y = sketch.random(20, sketch.height - 20);
            const hue = jewelHues[Math.floor(sketch.random(jewelHues.length))];
            points.push({ x, y, hue });
          }

          for (let i = 0; i < points.length; i++) {
            const p = points[i];
            const distFromCenter = sketch.dist(p.x, p.y, sketch.width / 2, sketch.height / 2);
            const normalizedDist = distFromCenter / (sketch.width * 0.7);
            const neighbors = countNeighbors(i);
            
            const saturation = sketch.map(neighbors, 2, 8, 70, 95);
            const brightness = sketch.map(normalizedDist, 0, 1, 85, 55);
            
            cellColors.set(i, {
              h: p.hue + sketch.random(-15, 15),
              s: saturation,
              b: brightness
            });
          }

          sketch.loadPixels();
          
          for (let y = 0; y < sketch.height; y++) {
            for (let x = 0; x < sketch.width; x++) {
              const idx = (y * sketch.width + x) * 4;
              
              if (isOnEdge(x, y)) {
                sketch.pixels[idx] = 30;
                sketch.pixels[idx + 1] = 30;
                sketch.pixels[idx + 2] = 30;
                sketch.pixels[idx + 3] = 255;
              } else {
                const closestIdx = findClosestPoint(x, y);
                const color = cellColors.get(closestIdx)!;
                
                const rgbColor = sketch.color(color.h, color.s, color.b);
                sketch.pixels[idx] = sketch.red(rgbColor);
                sketch.pixels[idx + 1] = sketch.green(rgbColor);
                sketch.pixels[idx + 2] = sketch.blue(rgbColor);
                sketch.pixels[idx + 3] = 255;
              }
            }
          }
          
          sketch.updatePixels();

          sketch.stroke(25);
          sketch.strokeWeight(leadingWeight);
          sketch.noFill();
          sketch.rect(0, 0, sketch.width, sketch.height);

          sketch.noLoop();
        };

        sketch.draw = () => {};
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
