<script lang="ts">
  import { onMount } from 'svelte';
  import p5 from 'p5';

  let container: HTMLDivElement;
  let p5Instance: p5 | null = null;

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        const seed = 12345;
        const bristleCount = 24;
        const strokeInterval = 60;
        const maxStrokes = 50;

        interface Bristle {
          offset: number;
          inkLoad: number;
          phase: number;
        }

        interface BrushStroke {
          startX: number;
          startY: number;
          endX: number;
          endY: number;
          controlX1: number;
          controlY1: number;
          controlX2: number;
          controlY2: number;
          bristles: Bristle[];
          progress: number;
          baseWidth: number;
          baseColor: { h: number; s: number; b: number };
          noiseOffset: number;
        }

        let strokes: BrushStroke[] = [];
        let frameCounter = 0;
        let strokeCount = 0;

        function createBristle(index: number): Bristle {
          return {
            offset: sketch.map(index, 0, bristleCount - 1, -1, 1),
            inkLoad: sketch.random(0.7, 1.0),
            phase: sketch.random(sketch.TWO_PI)
          };
        }

        function createStroke(): BrushStroke {
          const margin = 100;
          const startX = sketch.random(margin, sketch.width - margin);
          const startY = sketch.random(margin, sketch.height - margin);

          const angle = sketch.random(sketch.TWO_PI);
          const length = sketch.random(100, 300);
          const endX = startX + Math.cos(angle) * length;
          const endY = startY + Math.sin(angle) * length;

          const curvature = sketch.random(30, 80);
          const perpAngle = angle + sketch.HALF_PI;
          const curveDir = sketch.random() > 0.5 ? 1 : -1;

          const controlX1 = sketch.lerp(startX, endX, 0.33) + Math.cos(perpAngle) * curvature * curveDir;
          const controlY1 = sketch.lerp(startY, endY, 0.33) + Math.sin(perpAngle) * curvature * curveDir;
          const controlX2 = sketch.lerp(startX, endX, 0.66) + Math.cos(perpAngle) * curvature * curveDir * 0.5;
          const controlY2 = sketch.lerp(startY, endY, 0.66) + Math.sin(perpAngle) * curvature * curveDir * 0.5;

          const bristles: Bristle[] = [];
          for (let i = 0; i < bristleCount; i++) {
            bristles.push(createBristle(i));
          }

          const colorChoice = sketch.random();
          let baseColor: { h: number; s: number; b: number };
          if (colorChoice < 0.6) {
            baseColor = { h: 30, s: sketch.random(5, 20), b: sketch.random(5, 25) };
          } else if (colorChoice < 0.85) {
            baseColor = { h: sketch.random(20, 40), s: sketch.random(15, 35), b: sketch.random(25, 45) };
          } else {
            baseColor = { h: sketch.random(0, 20), s: sketch.random(30, 50), b: sketch.random(35, 55) };
          }

          return {
            startX,
            startY,
            endX,
            endY,
            controlX1,
            controlY1,
            controlX2,
            controlY2,
            bristles,
            progress: 0,
            baseWidth: sketch.random(15, 40),
            baseColor,
            noiseOffset: sketch.random(1000)
          };
        }

        function bezierPoint(t: number, p0: number, p1: number, p2: number, p3: number): number {
          const mt = 1 - t;
          return mt * mt * mt * p0 + 3 * mt * mt * t * p1 + 3 * mt * t * t * p2 + t * t * t * p3;
        }

        function bezierTangent(t: number, p0: number, p1: number, p2: number, p3: number): number {
          const mt = 1 - t;
          return 3 * mt * mt * (p1 - p0) + 6 * mt * t * (p2 - p1) + 3 * t * t * (p3 - p2);
        }

        function drawStrokeSegment(stroke: BrushStroke, t: number) {
          const x = bezierPoint(t, stroke.startX, stroke.controlX1, stroke.controlX2, stroke.endX);
          const y = bezierPoint(t, stroke.startY, stroke.controlY1, stroke.controlY2, stroke.endY);

          const tx = bezierTangent(t, stroke.startX, stroke.controlX1, stroke.controlX2, stroke.endX);
          const ty = bezierTangent(t, stroke.startY, stroke.controlY1, stroke.controlY2, stroke.endY);
          const perpX = -ty;
          const perpY = tx;
          const perpLen = Math.sqrt(perpX * perpX + perpY * perpY);
          const normPerpX = perpX / perpLen;
          const normPerpY = perpY / perpLen;

          const pressureNoise = sketch.noise(stroke.noiseOffset + t * 3);
          const pressure = 0.3 + pressureNoise * 0.7;
          const taperStart = t < 0.15 ? sketch.map(t, 0, 0.15, 0.2, 1) : 1;
          const taperEnd = t > 0.85 ? sketch.map(t, 0.85, 1, 1, 0.1) : 1;
          const width = stroke.baseWidth * pressure * taperStart * taperEnd;

          for (const bristle of stroke.bristles) {
            const bristlePhaseOffset = sketch.noise(stroke.noiseOffset + bristle.phase + t * 2) * 0.3;
            const bristleOffset = bristle.offset + bristlePhaseOffset;

            const inkRemaining = bristle.inkLoad * (1 - t * 0.6);
            if (inkRemaining < 0.1 && sketch.random() > inkRemaining * 5) continue;

            const bx = x + normPerpX * bristleOffset * width;
            const by = y + normPerpY * bristleOffset * width;

            const splay = sketch.map(pressure, 0.3, 1, 1.5, 0.8);
            const wobble = sketch.noise(stroke.noiseOffset + bristle.phase * 10 + t * 5) * 3 - 1.5;

            const finalX = bx + wobble * splay;
            const finalY = by + wobble * splay;

            const alpha = sketch.map(inkRemaining, 0, 1, 5, 40);
            const satMod = sketch.map(inkRemaining, 0, 1, -10, 5);
            const briMod = sketch.map(inkRemaining, 0, 1, 15, 0);

            sketch.stroke(
              stroke.baseColor.h,
              sketch.constrain(stroke.baseColor.s + satMod, 0, 100),
              sketch.constrain(stroke.baseColor.b + briMod, 0, 100),
              alpha
            );

            const strokeWeight = sketch.map(inkRemaining, 0, 1, 0.5, 2.5);
            sketch.strokeWeight(strokeWeight);
            sketch.point(finalX, finalY);
          }
        }

        sketch.setup = () => {
          sketch.randomSeed(seed);
          sketch.noiseSeed(seed);
          sketch.createCanvas(container.clientWidth, container.clientHeight);
          sketch.colorMode(sketch.HSB, 360, 100, 100, 100);
          sketch.background(40, 8, 96);
          sketch.noFill();
        };

        sketch.draw = () => {
          frameCounter++;

          if (frameCounter % strokeInterval === 0 && strokeCount < maxStrokes) {
            strokes.push(createStroke());
            strokeCount++;
          }

          for (const stroke of strokes) {
            if (stroke.progress < 1) {
              const segmentsPerFrame = 0.02;
              const startT = stroke.progress;
              const endT = Math.min(stroke.progress + segmentsPerFrame, 1);

              for (let t = startT; t < endT; t += 0.002) {
                drawStrokeSegment(stroke, t);
              }

              stroke.progress = endT;
            }
          }

          if (strokes.length > 0 && strokes.every((s) => s.progress >= 1) && strokeCount >= maxStrokes) {
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
