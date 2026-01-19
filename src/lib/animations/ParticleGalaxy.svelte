<script lang="ts">
  import { onMount } from 'svelte';
  import p5 from 'p5';

  let container: HTMLDivElement;
  let p5Instance: p5 | null = null;

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        const seed = 12345;
        const particleCount = 800;
        const centralMass = 5000;
        const spiralArms = 3;
        const spiralTightness = 0.3;
        const trailAlpha = 25;

        interface Particle {
          x: number;
          y: number;
          vx: number;
          vy: number;
          trail: { x: number; y: number }[];
          maxTrail: number;
        }

        let particles: Particle[] = [];
        let centerX: number;
        let centerY: number;

        function createParticle(): Particle {
          const armIndex = Math.floor(sketch.random(spiralArms));
          const armAngle = (armIndex / spiralArms) * sketch.TWO_PI;
          const distFromCenter = sketch.random(50, Math.min(sketch.width, sketch.height) * 0.45);
          const spiralOffset = distFromCenter * spiralTightness;
          const angle = armAngle + spiralOffset + sketch.random(-0.3, 0.3);

          const x = centerX + Math.cos(angle) * distFromCenter;
          const y = centerY + Math.sin(angle) * distFromCenter;

          const orbitalSpeed = Math.sqrt(centralMass / Math.max(distFromCenter, 10)) * 0.15;
          const tangentAngle = angle + sketch.HALF_PI;
          const vx = Math.cos(tangentAngle) * orbitalSpeed + sketch.random(-0.1, 0.1);
          const vy = Math.sin(tangentAngle) * orbitalSpeed + sketch.random(-0.1, 0.1);

          return {
            x,
            y,
            vx,
            vy,
            trail: [],
            maxTrail: Math.floor(sketch.random(5, 15))
          };
        }

        sketch.setup = () => {
          sketch.randomSeed(seed);
          sketch.noiseSeed(seed);
          sketch.createCanvas(container.clientWidth, container.clientHeight);
          sketch.colorMode(sketch.HSB, 360, 100, 100, 100);
          sketch.background(240, 20, 8);

          centerX = sketch.width / 2;
          centerY = sketch.height / 2;

          for (let i = 0; i < particleCount; i++) {
            particles.push(createParticle());
          }
        };

        sketch.draw = () => {
          sketch.background(240, 20, 8, 5);

          for (let i = 0; i < particles.length; i++) {
            const p = particles[i];

            p.trail.push({ x: p.x, y: p.y });
            if (p.trail.length > p.maxTrail) {
              p.trail.shift();
            }

            const dx = centerX - p.x;
            const dy = centerY - p.y;
            const distSq = dx * dx + dy * dy;
            const dist = Math.sqrt(distSq);

            const forceMag = centralMass / Math.max(distSq, 1000);
            const ax = (dx / dist) * forceMag * 0.01;
            const ay = (dy / dist) * forceMag * 0.01;

            p.vx += ax;
            p.vy += ay;

            p.x += p.vx;
            p.y += p.vy;

            const speed = Math.sqrt(p.vx * p.vx + p.vy * p.vy);
            const hue = sketch.map(speed, 0.5, 4, 20, 220);
            const saturation = sketch.map(dist, 50, 400, 80, 40);
            const brightness = sketch.map(speed, 0.5, 4, 50, 100);

            for (let j = 0; j < p.trail.length - 1; j++) {
              const alpha = sketch.map(j, 0, p.trail.length - 1, 5, trailAlpha);
              sketch.stroke(hue, saturation, brightness * 0.7, alpha);
              sketch.strokeWeight(1);
              sketch.line(p.trail[j].x, p.trail[j].y, p.trail[j + 1].x, p.trail[j + 1].y);
            }

            sketch.noStroke();
            sketch.fill(hue, saturation * 0.5, brightness, 80);
            const particleSize = sketch.map(speed, 0.5, 4, 2, 4);
            sketch.ellipse(p.x, p.y, particleSize, particleSize);

            if (
              p.x < -50 ||
              p.x > sketch.width + 50 ||
              p.y < -50 ||
              p.y > sketch.height + 50 ||
              dist < 5
            ) {
              particles[i] = createParticle();
            }
          }

          sketch.noStroke();
          const coreGradientSteps = 5;
          for (let r = coreGradientSteps; r >= 1; r--) {
            const radius = r * 8;
            const alpha = sketch.map(r, 1, coreGradientSteps, 60, 10);
            sketch.fill(45, 30, 100, alpha);
            sketch.ellipse(centerX, centerY, radius, radius);
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
