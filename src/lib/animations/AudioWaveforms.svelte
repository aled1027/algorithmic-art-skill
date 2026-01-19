<script lang="ts">
  import { onMount } from 'svelte';
  import p5 from 'p5';

  let container: HTMLDivElement;
  let p5Instance: p5 | null = null;

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        const seed = 12345;
        const numBands = 64;
        const smoothing = 0.85;
        const waveformResolution = 256;
        const radialRingCount = 5;
        const particleCount = 150;
        const beatThreshold = 0.65;
        const bassDecay = 0.92;
        const midDecay = 0.94;
        const highDecay = 0.96;

        let spectrum: number[] = [];
        let smoothedSpectrum: number[] = [];
        let waveform: number[] = [];
        let bassEnergy = 0;
        let midEnergy = 0;
        let highEnergy = 0;
        let beatPulse = 0;
        let particles: Particle[] = [];
        let time = 0;

        interface Particle {
          angle: number;
          radius: number;
          baseRadius: number;
          speed: number;
          size: number;
          hue: number;
          life: number;
          maxLife: number;
        }

        function simulateSpectrum(): number[] {
          const result: number[] = [];
          const t = time * 0.02;

          for (let i = 0; i < numBands; i++) {
            const freq = i / numBands;
            let value = 0;

            value += (1 - freq) * 0.6 * (0.5 + 0.5 * sketch.sin(t * 2.3 + sketch.noise(i * 0.1, t) * 4));
            value += (freq > 0.1 && freq < 0.5 ? 1 : 0) * 0.4 * (0.5 + 0.5 * sketch.sin(t * 3.7 + i * 0.2));
            value += freq * 0.5 * sketch.noise(i * 0.3, t * 2);
            value += sketch.noise(i * 0.05, t * 0.5) * 0.3;

            const beatNoise = sketch.noise(t * 0.3);
            if (beatNoise > beatThreshold && i < 8) {
              value += (beatThreshold - beatNoise + 0.35) * 2;
            }

            result.push(sketch.constrain(value, 0, 1));
          }
          return result;
        }

        function simulateWaveform(): number[] {
          const result: number[] = [];
          const t = time * 0.03;

          for (let i = 0; i < waveformResolution; i++) {
            const x = i / waveformResolution;
            let y = 0;

            y += sketch.sin(x * sketch.TWO_PI * 2 + t * 4) * bassEnergy * 0.4;
            y += sketch.sin(x * sketch.TWO_PI * 5 + t * 7) * midEnergy * 0.3;
            y += sketch.sin(x * sketch.TWO_PI * 12 + t * 11) * highEnergy * 0.2;
            y += sketch.noise(x * 3, t) * 0.15 - 0.075;

            result.push(y);
          }
          return result;
        }

        function calculateEnergies() {
          const bassRange = Math.floor(numBands * 0.15);
          const midRange = Math.floor(numBands * 0.5);

          let bassSum = 0;
          let midSum = 0;
          let highSum = 0;

          for (let i = 0; i < bassRange; i++) {
            bassSum += smoothedSpectrum[i] || 0;
          }
          for (let i = bassRange; i < midRange; i++) {
            midSum += smoothedSpectrum[i] || 0;
          }
          for (let i = midRange; i < numBands; i++) {
            highSum += smoothedSpectrum[i] || 0;
          }

          const targetBass = bassSum / bassRange;
          const targetMid = midSum / (midRange - bassRange);
          const targetHigh = highSum / (numBands - midRange);

          bassEnergy = bassEnergy * bassDecay + targetBass * (1 - bassDecay);
          midEnergy = midEnergy * midDecay + targetMid * (1 - midDecay);
          highEnergy = highEnergy * highDecay + targetHigh * (1 - highDecay);

          if (targetBass > beatThreshold && bassEnergy > 0.5) {
            beatPulse = 1;
            spawnBeatParticles();
          }
          beatPulse *= 0.9;
        }

        function createParticle(fromBeat = false): Particle {
          const baseRadius = fromBeat
            ? sketch.random(80, 150)
            : sketch.random(100, Math.min(sketch.width, sketch.height) * 0.35);
          return {
            angle: sketch.random(sketch.TWO_PI),
            radius: baseRadius,
            baseRadius,
            speed: sketch.random(0.005, 0.02) * (fromBeat ? 2 : 1),
            size: sketch.random(2, 6),
            hue: fromBeat ? sketch.random(0, 60) : sketch.random(180, 300),
            life: 0,
            maxLife: sketch.random(60, 180)
          };
        }

        function spawnBeatParticles() {
          for (let i = 0; i < 8; i++) {
            particles.push(createParticle(true));
          }
        }

        function drawRadialSpectrum() {
          const cx = sketch.width / 2;
          const cy = sketch.height / 2;
          const baseRadius = Math.min(sketch.width, sketch.height) * 0.15;
          const maxRadius = Math.min(sketch.width, sketch.height) * 0.4;

          for (let ring = 0; ring < radialRingCount; ring++) {
            const ringRadius = baseRadius + (ring / radialRingCount) * (maxRadius - baseRadius);
            const ringAlpha = sketch.map(ring, 0, radialRingCount, 80, 30);
            const bandOffset = Math.floor((ring / radialRingCount) * numBands * 0.3);

            sketch.beginShape();
            sketch.noFill();

            for (let i = 0; i <= numBands; i++) {
              const idx = (i + bandOffset) % numBands;
              const angle = (i / numBands) * sketch.TWO_PI - sketch.HALF_PI;
              const amp = smoothedSpectrum[idx] || 0;
              const r = ringRadius + amp * 80 * (1 + beatPulse * 0.3);

              const hue = sketch.map(idx, 0, numBands, 0, 270);
              const sat = 70 + amp * 30;
              const bri = 50 + amp * 50;

              sketch.stroke(hue, sat, bri, ringAlpha);
              sketch.strokeWeight(2 + amp * 2);

              const x = cx + sketch.cos(angle) * r;
              const y = cy + sketch.sin(angle) * r;
              sketch.vertex(x, y);
            }
            sketch.endShape(sketch.CLOSE);
          }
        }

        function drawOscilloscope() {
          const cx = sketch.width / 2;
          const cy = sketch.height / 2;
          const waveWidth = sketch.width * 0.8;
          const waveHeight = 100;

          sketch.noFill();
          sketch.strokeWeight(2);

          for (let layer = 0; layer < 3; layer++) {
            const offset = layer * 30;
            const alpha = 60 - layer * 15;
            const hue = layer === 0 ? 200 : layer === 1 ? 280 : 320;

            sketch.beginShape();
            for (let i = 0; i < waveformResolution; i++) {
              const x = cx - waveWidth / 2 + (i / waveformResolution) * waveWidth;
              const y = cy + sketch.height * 0.35 + waveform[i] * waveHeight + offset;

              const localAmp = Math.abs(waveform[i]);
              sketch.stroke(hue, 60 + localAmp * 40, 70 + localAmp * 30, alpha);
              sketch.vertex(x, y);
            }
            sketch.endShape();
          }
        }

        function drawCenterGlow() {
          const cx = sketch.width / 2;
          const cy = sketch.height / 2;
          const pulseSize = 60 + bassEnergy * 40 + beatPulse * 30;

          for (let r = pulseSize; r > 0; r -= 5) {
            const alpha = sketch.map(r, 0, pulseSize, 40, 0);
            const hue = sketch.map(r, 0, pulseSize, 30, 280);
            sketch.noStroke();
            sketch.fill(hue, 70, 80, alpha);
            sketch.ellipse(cx, cy, r * 2, r * 2);
          }
        }

        function updateParticles() {
          for (let i = particles.length - 1; i >= 0; i--) {
            const p = particles[i];
            p.angle += p.speed;
            p.radius += (bassEnergy - 0.3) * 2;
            p.life++;

            if (p.life > p.maxLife || p.radius < 20 || p.radius > sketch.width) {
              particles.splice(i, 1);
              continue;
            }

            const lifeRatio = p.life / p.maxLife;
            const alpha = (1 - lifeRatio) * 60;
            const cx = sketch.width / 2;
            const cy = sketch.height / 2;
            const x = cx + sketch.cos(p.angle) * p.radius;
            const y = cy + sketch.sin(p.angle) * p.radius;

            sketch.noStroke();
            sketch.fill(p.hue, 70, 90, alpha);
            sketch.ellipse(x, y, p.size * (1 + bassEnergy), p.size * (1 + bassEnergy));
          }
        }

        function drawFrequencyBars() {
          const barWidth = sketch.width / numBands;
          const maxHeight = sketch.height * 0.15;
          const y = sketch.height - 10;

          for (let i = 0; i < numBands; i++) {
            const amp = smoothedSpectrum[i] || 0;
            const h = amp * maxHeight;
            const hue = sketch.map(i, 0, numBands, 0, 270);

            sketch.noStroke();
            sketch.fill(hue, 70, 80, 50);
            sketch.rect(i * barWidth + 1, y - h, barWidth - 2, h, 2);
          }
        }

        sketch.setup = () => {
          sketch.randomSeed(seed);
          sketch.noiseSeed(seed);
          sketch.createCanvas(container.clientWidth, container.clientHeight);
          sketch.colorMode(sketch.HSB, 360, 100, 100, 100);
          sketch.background(0, 0, 5);

          smoothedSpectrum = new Array(numBands).fill(0);
          waveform = new Array(waveformResolution).fill(0);

          for (let i = 0; i < particleCount; i++) {
            particles.push(createParticle());
          }
        };

        sketch.draw = () => {
          sketch.background(0, 0, 5, 25);
          time++;

          spectrum = simulateSpectrum();

          for (let i = 0; i < numBands; i++) {
            smoothedSpectrum[i] = smoothedSpectrum[i] * smoothing + spectrum[i] * (1 - smoothing);
          }

          calculateEnergies();
          waveform = simulateWaveform();

          drawCenterGlow();
          drawRadialSpectrum();
          updateParticles();
          drawOscilloscope();
          drawFrequencyBars();

          while (particles.length < particleCount) {
            particles.push(createParticle());
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
