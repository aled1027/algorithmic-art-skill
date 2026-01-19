<script lang="ts">
  import { onMount } from 'svelte';
  import p5 from 'p5';

  let container: HTMLDivElement;
  let p5Instance: p5 | null = null;

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        const seed = 31415;
        const cellSize = 6;
        const initialDensity = 0.35;
        const fadeSpeed = 0.92;

        let cols: number;
        let rows: number;
        let grid: number[][];
        let nextGrid: number[][];
        let ages: number[][];

        function countNeighbors(x: number, y: number): number {
          let count = 0;
          for (let i = -1; i <= 1; i++) {
            for (let j = -1; j <= 1; j++) {
              if (i === 0 && j === 0) continue;
              const nx = (x + i + cols) % cols;
              const ny = (y + j + rows) % rows;
              count += grid[nx][ny] > 0.5 ? 1 : 0;
            }
          }
          return count;
        }

        sketch.setup = () => {
          sketch.randomSeed(seed);
          sketch.noiseSeed(seed);
          sketch.createCanvas(container.clientWidth, container.clientHeight);
          sketch.colorMode(sketch.HSB, 360, 100, 100, 100);

          cols = Math.floor(sketch.width / cellSize);
          rows = Math.floor(sketch.height / cellSize);

          grid = Array(cols)
            .fill(null)
            .map(() => Array(rows).fill(0));
          nextGrid = Array(cols)
            .fill(null)
            .map(() => Array(rows).fill(0));
          ages = Array(cols)
            .fill(null)
            .map(() => Array(rows).fill(0));

          for (let x = 0; x < cols; x++) {
            for (let y = 0; y < rows; y++) {
              const noiseVal = sketch.noise(x * 0.1, y * 0.1);
              grid[x][y] = noiseVal < initialDensity ? 1 : 0;
            }
          }
        };

        sketch.draw = () => {
          sketch.background(240, 10, 15);

          for (let x = 0; x < cols; x++) {
            for (let y = 0; y < rows; y++) {
              const neighbors = countNeighbors(x, y);
              const alive = grid[x][y] > 0.5;

              if (alive) {
                nextGrid[x][y] = neighbors === 2 || neighbors === 3 ? 1 : 0;
              } else {
                nextGrid[x][y] = neighbors === 3 ? 1 : grid[x][y] * fadeSpeed;
              }

              if (nextGrid[x][y] > 0.5 && grid[x][y] > 0.5) {
                ages[x][y]++;
              } else if (nextGrid[x][y] > 0.5) {
                ages[x][y] = 0;
              }
            }
          }

          for (let x = 0; x < cols; x++) {
            for (let y = 0; y < rows; y++) {
              const value = grid[x][y];
              if (value > 0.05) {
                const age = ages[x][y];
                const hue = value > 0.5 ? (180 + age * 2) % 360 : 280;
                const saturation = value > 0.5 ? 70 : 40;
                const brightness = sketch.map(value, 0, 1, 20, 90);

                sketch.fill(hue, saturation, brightness);
                sketch.noStroke();
                sketch.rect(x * cellSize, y * cellSize, cellSize - 1, cellSize - 1);
              }
            }
          }

          const temp = grid;
          grid = nextGrid;
          nextGrid = temp;
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
