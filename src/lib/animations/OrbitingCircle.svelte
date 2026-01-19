<script lang="ts">
  import { onMount } from "svelte"
  import p5 from "p5"

  let container: HTMLDivElement
  let p5Instance: p5 | null = null

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        sketch.setup = () => {
          sketch.createCanvas(400, 300)
        }

        sketch.draw = () => {
          sketch.background(220)
          sketch.fill(100, 150, 255)
          sketch.noStroke()
          sketch.ellipse(
            sketch.width / 2 + sketch.sin(sketch.frameCount * 0.05) * 100,
            sketch.height / 2 + sketch.cos(sketch.frameCount * 0.05) * 100,
            50,
            50,
          )
        }
      }, container)
    }

    return () => {
      if (p5Instance) {
        p5Instance.remove()
      }
    }
  })
</script>

<div bind:this={container} class="overflow-hidden rounded-lg shadow-lg"></div>
