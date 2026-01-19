<script lang="ts">
  import { onMount } from "svelte"
  import p5 from "p5"

  let container: HTMLDivElement
  let p5Instance: p5 | null = null

  onMount(() => {
    if (container) {
      p5Instance = new p5((sketch: p5) => {
        sketch.setup = () => {
          sketch.createCanvas(container.clientWidth, container.clientHeight)
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

<div class="container" bind:this={container}></div>

<style>
  .container {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
  }
</style>


