<script lang="ts">
  import { onMount } from 'svelte';
  import p5 from 'p5';

  let container: HTMLDivElement;
  let p5Instance: p5 | null = null;

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        const seed = 12345;
        const maxDepth = 8;
        const initialLength = 180;
        const lengthRatio = 0.67;
        const baseAngle = sketch.PI / 6;
        const noiseScale = 0.3;
        const trunkWeight = 12;

        interface BranchSegment {
          x1: number;
          y1: number;
          x2: number;
          y2: number;
          depth: number;
          angle: number;
        }

        let branches: BranchSegment[] = [];
        let currentBranch = 0;
        let isGrowing = true;

        function generateTree(
          x: number,
          y: number,
          length: number,
          angle: number,
          depth: number
        ) {
          if (depth > maxDepth || length < 2) return;

          const noiseVal = sketch.noise(x * 0.01, y * 0.01, depth * 0.5);
          const angleOffset = sketch.map(noiseVal, 0, 1, -noiseScale, noiseScale);

          const adjustedAngle = angle + angleOffset;
          const x2 = x + Math.cos(adjustedAngle) * length;
          const y2 = y + Math.sin(adjustedAngle) * length;

          branches.push({
            x1: x,
            y1: y,
            x2: x2,
            y2: y2,
            depth: depth,
            angle: adjustedAngle
          });

          const newLength = length * lengthRatio;
          const leftAngle = adjustedAngle - baseAngle + sketch.random(-0.1, 0.1);
          const rightAngle = adjustedAngle + baseAngle + sketch.random(-0.1, 0.1);

          generateTree(x2, y2, newLength, leftAngle, depth + 1);
          generateTree(x2, y2, newLength, rightAngle, depth + 1);

          if (depth > 3 && sketch.random() > 0.6) {
            const midAngle = adjustedAngle + sketch.random(-0.3, 0.3);
            generateTree(x2, y2, newLength * 0.8, midAngle, depth + 1);
          }
        }

        function getBranchColor(depth: number): p5.Color {
          const t = depth / maxDepth;
          if (t < 0.4) {
            return sketch.lerpColor(
              sketch.color(60, 40, 20),
              sketch.color(80, 55, 30),
              t / 0.4
            );
          } else {
            return sketch.lerpColor(
              sketch.color(80, 55, 30),
              sketch.color(45, 120, 50),
              (t - 0.4) / 0.6
            );
          }
        }

        function getBranchWeight(depth: number): number {
          const t = depth / maxDepth;
          return sketch.lerp(trunkWeight, 1, t * t);
        }

        sketch.setup = () => {
          sketch.randomSeed(seed);
          sketch.noiseSeed(seed);
          sketch.createCanvas(container.clientWidth, container.clientHeight);
          sketch.background(245, 240, 230);

          const startX = sketch.width / 2;
          const startY = sketch.height - 50;
          generateTree(startX, startY, initialLength, -sketch.HALF_PI, 0);
        };

        sketch.draw = () => {
          if (!isGrowing) return;

          const branchesPerFrame = 5;
          for (let i = 0; i < branchesPerFrame && currentBranch < branches.length; i++) {
            const branch = branches[currentBranch];

            sketch.stroke(getBranchColor(branch.depth));
            sketch.strokeWeight(getBranchWeight(branch.depth));
            sketch.strokeCap(sketch.ROUND);
            sketch.line(branch.x1, branch.y1, branch.x2, branch.y2);

            if (branch.depth >= maxDepth - 1 && sketch.random() > 0.3) {
              const leafSize = sketch.random(3, 6);
              const leafOffset = sketch.random(0.3, 1);
              const leafX = branch.x2 + Math.cos(branch.angle) * leafSize * leafOffset;
              const leafY = branch.y2 + Math.sin(branch.angle) * leafSize * leafOffset;

              sketch.noStroke();
              const leafHue = sketch.random() > 0.5
                ? sketch.color(50, 140, 50, 180)
                : sketch.color(80, 160, 60, 180);
              sketch.fill(leafHue);
              sketch.ellipse(leafX, leafY, leafSize, leafSize * 1.5);
            }

            currentBranch++;
          }

          if (currentBranch >= branches.length) {
            isGrowing = false;
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
