<script lang="ts">
  import { onMount } from 'svelte';
  import p5 from 'p5';

  let container: HTMLDivElement;
  let p5Instance: p5 | null = null;

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        const seed = 42;
        const particleCount = 800;
        const noiseScale = 0.005;
        const particleSpeed = 2;
        const trailAlpha = 15;

        interface Particle {
          x: number;
          y: number;
          prevX: number;
          prevY: number;
          life: number;
          maxLife: number;
          hue: number;
        }

        let particles: Particle[] = [];
        let flowField: number[][] = [];
        let cols: number;
        let rows: number;
        const resolution = 10;

        function createParticle(): Particle {
          const x = sketch.random(sketch.width);
          const y = sketch.random(sketch.height);
          return {
            x,
            y,
            prevX: x,
            prevY: y,
            life: 0,
            maxLife: sketch.random(100, 300),
            hue: sketch.map(y, 0, sketch.height, 30, 200)
          };
        }

        function updateFlowField() {
          for (let y = 0; y < rows; y++) {
            for (let x = 0; x < cols; x++) {
              const angle =
                sketch.noise(x * noiseScale * 10, y * noiseScale * 10, sketch.frameCount * 0.002) *
                sketch.TWO_PI *
                2;
              flowField[y][x] = angle;
            }
          }
        }

        sketch.setup = () => {
          sketch.randomSeed(seed);
          sketch.noiseSeed(seed);
          sketch.createCanvas(container.clientWidth, container.clientHeight);
          sketch.colorMode(sketch.HSB, 360, 100, 100, 100);
          sketch.background(220, 15, 95);

          cols = Math.ceil(sketch.width / resolution);
          rows = Math.ceil(sketch.height / resolution);

          flowField = Array(rows)
            .fill(null)
            .map(() => Array(cols).fill(0));

          for (let i = 0; i < particleCount; i++) {
            particles.push(createParticle());
          }
        };

        sketch.draw = () => {
          updateFlowField();

          for (let i = 0; i < particles.length; i++) {
            const p = particles[i];
            p.prevX = p.x;
            p.prevY = p.y;

            const col = Math.floor(p.x / resolution);
            const row = Math.floor(p.y / resolution);

            if (col >= 0 && col < cols && row >= 0 && row < rows) {
              const angle = flowField[row][col];
              p.x += Math.cos(angle) * particleSpeed;
              p.y += Math.sin(angle) * particleSpeed;
            }

            p.life++;

            if (p.x < 0 || p.x > sketch.width || p.y < 0 || p.y > sketch.height || p.life > p.maxLife) {
              particles[i] = createParticle();
            } else {
              const velocity = sketch.dist(p.prevX, p.prevY, p.x, p.y);
              const saturation = sketch.map(velocity, 0, particleSpeed * 2, 20, 80);
              const brightness = sketch.map(velocity, 0, particleSpeed * 2, 40, 90);

              sketch.stroke(p.hue, saturation, brightness, trailAlpha);
              sketch.strokeWeight(1);
              sketch.line(p.prevX, p.prevY, p.x, p.y);
            }
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
