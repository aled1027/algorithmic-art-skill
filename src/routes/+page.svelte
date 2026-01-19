<script lang="ts">
	import { onMount } from 'svelte';
	import p5 from 'p5';

	let container: HTMLDivElement;
	let p5Instance: p5 | null = null;

	onMount(() => {
		if (container) {
			p5Instance = new p5((sketch: p5) => {
				sketch.setup = () => {
					sketch.createCanvas(800, 600);
				};

				sketch.draw = () => {
					sketch.background(220);
					sketch.fill(100, 150, 255);
					sketch.noStroke();
					sketch.ellipse(
						sketch.width / 2 + sketch.sin(sketch.frameCount * 0.05) * 100,
						sketch.height / 2 + sketch.cos(sketch.frameCount * 0.05) * 100,
						50,
						50
					);
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

<h1 class="text-blue-500">Welcome to SvelteKit</h1>
<p>Visit <a href="https://svelte.dev/docs/kit">svelte.dev/docs/kit</a> to read the documentation</p>

<div bind:this={container}></div>
