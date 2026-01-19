<script lang="ts">
  import { onMount } from 'svelte';
  import p5 from 'p5';

  let container: HTMLDivElement;
  let p5Instance: p5 | null = null;

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        const seed = 42;
        const contourLevels = 10;
        const noiseScale = 0.008;
        const noiseOctaves = 4;

        interface ElevationColor {
          elevation: number;
          color: [number, number, number];
        }

        const elevationPalette: ElevationColor[] = [
          { elevation: 0.0, color: [20, 50, 120] },
          { elevation: 0.2, color: [30, 90, 160] },
          { elevation: 0.35, color: [60, 160, 190] },
          { elevation: 0.4, color: [210, 190, 140] },
          { elevation: 0.45, color: [100, 170, 80] },
          { elevation: 0.55, color: [60, 130, 50] },
          { elevation: 0.7, color: [90, 70, 50] },
          { elevation: 0.85, color: [140, 120, 100] },
          { elevation: 1.0, color: [250, 250, 255] }
        ];

        function getElevationColor(elevation: number): [number, number, number] {
          for (let i = 0; i < elevationPalette.length - 1; i++) {
            const current = elevationPalette[i];
            const next = elevationPalette[i + 1];
            if (elevation >= current.elevation && elevation <= next.elevation) {
              const t = (elevation - current.elevation) / (next.elevation - current.elevation);
              return [
                sketch.lerp(current.color[0], next.color[0], t),
                sketch.lerp(current.color[1], next.color[1], t),
                sketch.lerp(current.color[2], next.color[2], t)
              ];
            }
          }
          return elevationPalette[elevationPalette.length - 1].color;
        }

        function octaveNoise(x: number, y: number): number {
          let value = 0;
          let amplitude = 1;
          let frequency = 1;
          let maxValue = 0;

          for (let i = 0; i < noiseOctaves; i++) {
            value += sketch.noise(x * frequency, y * frequency) * amplitude;
            maxValue += amplitude;
            amplitude *= 0.5;
            frequency *= 2;
          }

          return value / maxValue;
        }

        sketch.setup = () => {
          sketch.randomSeed(seed);
          sketch.noiseSeed(seed);
          sketch.createCanvas(container.clientWidth, container.clientHeight);

          const elevationGrid: number[][] = [];
          for (let y = 0; y < sketch.height; y++) {
            elevationGrid[y] = [];
            for (let x = 0; x < sketch.width; x++) {
              elevationGrid[y][x] = octaveNoise(x * noiseScale, y * noiseScale);
            }
          }

          sketch.loadPixels();
          for (let y = 0; y < sketch.height; y++) {
            for (let x = 0; x < sketch.width; x++) {
              const elevation = elevationGrid[y][x];
              const [r, g, b] = getElevationColor(elevation);
              const index = (y * sketch.width + x) * 4;
              sketch.pixels[index] = r;
              sketch.pixels[index + 1] = g;
              sketch.pixels[index + 2] = b;
              sketch.pixels[index + 3] = 255;
            }
          }
          sketch.updatePixels();

          sketch.stroke(40, 30, 20);
          sketch.noFill();

          for (let level = 0; level <= contourLevels; level++) {
            const threshold = level / contourLevels;
            const isMajor = level % 2 === 0;
            sketch.strokeWeight(isMajor ? 1.2 : 0.6);
            sketch.stroke(40, 30, 20, isMajor ? 180 : 100);

            for (let y = 0; y < sketch.height - 1; y += 2) {
              for (let x = 0; x < sketch.width - 1; x += 2) {
                const tl = elevationGrid[y][x];
                const tr = elevationGrid[y][x + 2] ?? elevationGrid[y][x + 1] ?? tl;
                const bl = elevationGrid[y + 2]?.[x] ?? elevationGrid[y + 1]?.[x] ?? tl;
                const br = elevationGrid[y + 2]?.[x + 2] ?? elevationGrid[y + 1]?.[x + 1] ?? tl;

                const cellCase =
                  (tl >= threshold ? 8 : 0) +
                  (tr >= threshold ? 4 : 0) +
                  (br >= threshold ? 2 : 0) +
                  (bl >= threshold ? 1 : 0);

                if (cellCase === 0 || cellCase === 15) continue;

                const cellSize = 2;
                const topInterp = (threshold - tl) / (tr - tl);
                const rightInterp = (threshold - tr) / (br - tr);
                const bottomInterp = (threshold - bl) / (br - bl);
                const leftInterp = (threshold - tl) / (bl - tl);

                const top = { x: x + topInterp * cellSize, y: y };
                const right = { x: x + cellSize, y: y + rightInterp * cellSize };
                const bottom = { x: x + bottomInterp * cellSize, y: y + cellSize };
                const left = { x: x, y: y + leftInterp * cellSize };

                switch (cellCase) {
                  case 1:
                  case 14:
                    sketch.line(left.x, left.y, bottom.x, bottom.y);
                    break;
                  case 2:
                  case 13:
                    sketch.line(bottom.x, bottom.y, right.x, right.y);
                    break;
                  case 3:
                  case 12:
                    sketch.line(left.x, left.y, right.x, right.y);
                    break;
                  case 4:
                  case 11:
                    sketch.line(top.x, top.y, right.x, right.y);
                    break;
                  case 5:
                    sketch.line(left.x, left.y, top.x, top.y);
                    sketch.line(bottom.x, bottom.y, right.x, right.y);
                    break;
                  case 6:
                  case 9:
                    sketch.line(top.x, top.y, bottom.x, bottom.y);
                    break;
                  case 7:
                  case 8:
                    sketch.line(left.x, left.y, top.x, top.y);
                    break;
                  case 10:
                    sketch.line(left.x, left.y, bottom.x, bottom.y);
                    sketch.line(top.x, top.y, right.x, right.y);
                    break;
                }
              }
            }
          }

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
