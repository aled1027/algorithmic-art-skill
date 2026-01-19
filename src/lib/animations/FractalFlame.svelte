<script lang="ts">
  import { onMount } from 'svelte';
  import p5 from 'p5';

  let container: HTMLDivElement;
  let p5Instance: p5 | null = null;

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        const seed = 12345;
        const iterationsPerFrame = 5000;
        const gamma = 2.2;
        const numTransforms = 4;

        interface Transform {
          a: number;
          b: number;
          c: number;
          d: number;
          e: number;
          f: number;
          weight: number;
          colorIndex: number;
          variations: { type: string; weight: number }[];
        }

        let transforms: Transform[] = [];
        let histogram: number[][] = [];
        let colorAccum: number[][][] = [];
        let maxDensity = 0;
        let x = 0;
        let y = 0;
        let colorValue = 0;
        let histWidth: number;
        let histHeight: number;
        let totalWeight: number;
        let palette: { r: number; g: number; b: number }[] = [];

        function seededRandom(): number {
          return sketch.random();
        }

        function createTransform(): Transform {
          const a = seededRandom() * 1.6 - 0.8;
          const b = seededRandom() * 1.6 - 0.8;
          const c = seededRandom() * 1.6 - 0.8;
          const d = seededRandom() * 1.6 - 0.8;
          const e = seededRandom() * 2 - 1;
          const f = seededRandom() * 2 - 1;

          const variations: { type: string; weight: number }[] = [];
          const variationTypes = ['linear', 'sinusoidal', 'spherical', 'swirl', 'horseshoe', 'polar'];
          const numVars = Math.floor(seededRandom() * 3) + 1;
          let totalVarWeight = 0;

          for (let i = 0; i < numVars; i++) {
            const type = variationTypes[Math.floor(seededRandom() * variationTypes.length)];
            const weight = seededRandom();
            totalVarWeight += weight;
            variations.push({ type, weight });
          }

          for (const v of variations) {
            v.weight /= totalVarWeight;
          }

          return {
            a,
            b,
            c,
            d,
            e,
            f,
            weight: seededRandom() * 0.8 + 0.2,
            colorIndex: seededRandom(),
            variations
          };
        }

        function applyVariation(
          type: string,
          ax: number,
          ay: number
        ): { x: number; y: number } {
          const r2 = ax * ax + ay * ay;
          const r = Math.sqrt(r2);
          const theta = Math.atan2(ay, ax);

          switch (type) {
            case 'linear':
              return { x: ax, y: ay };
            case 'sinusoidal':
              return { x: Math.sin(ax), y: Math.sin(ay) };
            case 'spherical':
              if (r2 < 0.0001) return { x: 0, y: 0 };
              return { x: ax / r2, y: ay / r2 };
            case 'swirl':
              return {
                x: ax * Math.sin(r2) - ay * Math.cos(r2),
                y: ax * Math.cos(r2) + ay * Math.sin(r2)
              };
            case 'horseshoe':
              if (r < 0.0001) return { x: 0, y: 0 };
              return {
                x: ((ax - ay) * (ax + ay)) / r,
                y: (2 * ax * ay) / r
              };
            case 'polar':
              return { x: theta / Math.PI, y: r - 1 };
            default:
              return { x: ax, y: ay };
          }
        }

        function applyTransform(t: Transform, px: number, py: number): { x: number; y: number } {
          const ax = t.a * px + t.b * py + t.e;
          const ay = t.c * px + t.d * py + t.f;

          let resultX = 0;
          let resultY = 0;

          for (const v of t.variations) {
            const result = applyVariation(v.type, ax, ay);
            resultX += result.x * v.weight;
            resultY += result.y * v.weight;
          }

          return { x: resultX, y: resultY };
        }

        function selectTransform(): Transform {
          let r = seededRandom() * totalWeight;
          for (const t of transforms) {
            r -= t.weight;
            if (r <= 0) return t;
          }
          return transforms[transforms.length - 1];
        }

        function worldToScreen(wx: number, wy: number): { sx: number; sy: number } {
          const scale = Math.min(histWidth, histHeight) * 0.4;
          const sx = Math.floor(wx * scale + histWidth / 2);
          const sy = Math.floor(wy * scale + histHeight / 2);
          return { sx, sy };
        }

        sketch.setup = () => {
          sketch.randomSeed(seed);
          sketch.noiseSeed(seed);
          sketch.createCanvas(container.clientWidth, container.clientHeight);
          sketch.pixelDensity(1);
          sketch.background(0);

          histWidth = sketch.width;
          histHeight = sketch.height;

          histogram = [];
          colorAccum = [];
          for (let i = 0; i < histWidth; i++) {
            histogram[i] = [];
            colorAccum[i] = [];
            for (let j = 0; j < histHeight; j++) {
              histogram[i][j] = 0;
              colorAccum[i][j] = [0, 0, 0];
            }
          }

          palette = [
            { r: 255, g: 60, b: 20 },
            { r: 255, g: 180, b: 40 },
            { r: 200, g: 80, b: 255 },
            { r: 60, g: 200, b: 255 },
            { r: 255, g: 255, b: 200 }
          ];

          transforms = [];
          for (let i = 0; i < numTransforms; i++) {
            transforms.push(createTransform());
          }

          totalWeight = transforms.reduce((sum, t) => sum + t.weight, 0);

          x = seededRandom() * 2 - 1;
          y = seededRandom() * 2 - 1;
          colorValue = seededRandom();

          for (let i = 0; i < 20; i++) {
            const t = selectTransform();
            const result = applyTransform(t, x, y);
            x = result.x;
            y = result.y;
            colorValue = (colorValue + t.colorIndex) / 2;
          }
        };

        sketch.draw = () => {
          for (let iter = 0; iter < iterationsPerFrame; iter++) {
            const t = selectTransform();
            const result = applyTransform(t, x, y);
            x = result.x;
            y = result.y;
            colorValue = (colorValue + t.colorIndex) / 2;

            if (!isFinite(x) || !isFinite(y) || Math.abs(x) > 10 || Math.abs(y) > 10) {
              x = seededRandom() * 2 - 1;
              y = seededRandom() * 2 - 1;
              colorValue = seededRandom();
              continue;
            }

            const { sx, sy } = worldToScreen(x, y);

            if (sx >= 0 && sx < histWidth && sy >= 0 && sy < histHeight) {
              histogram[sx][sy]++;
              if (histogram[sx][sy] > maxDensity) {
                maxDensity = histogram[sx][sy];
              }

              const paletteIndex = colorValue * (palette.length - 1);
              const idx1 = Math.floor(paletteIndex);
              const idx2 = Math.min(idx1 + 1, palette.length - 1);
              const frac = paletteIndex - idx1;

              const r = palette[idx1].r * (1 - frac) + palette[idx2].r * frac;
              const g = palette[idx1].g * (1 - frac) + palette[idx2].g * frac;
              const b = palette[idx1].b * (1 - frac) + palette[idx2].b * frac;

              colorAccum[sx][sy][0] += r;
              colorAccum[sx][sy][1] += g;
              colorAccum[sx][sy][2] += b;
            }
          }

          if (sketch.frameCount % 10 === 0 && maxDensity > 0) {
            sketch.loadPixels();
            const logMaxDensity = Math.log(maxDensity + 1);

            for (let i = 0; i < histWidth; i++) {
              for (let j = 0; j < histHeight; j++) {
                const density = histogram[i][j];
                if (density === 0) continue;

                const logDensity = Math.log(density + 1) / logMaxDensity;
                const alpha = Math.pow(logDensity, 1 / gamma);

                const r = (colorAccum[i][j][0] / density) * alpha;
                const g = (colorAccum[i][j][1] / density) * alpha;
                const b = (colorAccum[i][j][2] / density) * alpha;

                const pixelIndex = (j * histWidth + i) * 4;
                sketch.pixels[pixelIndex] = Math.min(255, r);
                sketch.pixels[pixelIndex + 1] = Math.min(255, g);
                sketch.pixels[pixelIndex + 2] = Math.min(255, b);
                sketch.pixels[pixelIndex + 3] = 255;
              }
            }
            sketch.updatePixels();
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
