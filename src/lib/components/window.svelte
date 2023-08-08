<script lang="ts">
	import { SvelteComponent, onMount } from "svelte"
	import type { ComponentType } from "svelte"
	import { spring } from "svelte/motion"
	import { fade } from "svelte/transition"
	import { pannable } from "$lib/utils/pannable.js"
	import { windows } from "$lib/utils/stores"

	export let id: string,
		name: string,
		content: ComponentType<SvelteComponent>,
		w: number,
		h: number,
		posX: number,
		posY: number

	let width: number, height: number

	$: width, height && setTimeout(checkOverflow, 300)

	// Calculate the size of the window
	$: size = [(width * w) / 1920, (height * h) / 920]

	// Min window width is 300px
	$: size[0] = Math.max(300, size[0])
	// Min window height is 200px
	$: size[1] = Math.max(200, size[1])

	const coords = spring(
		{ x: 0, y: 0 },
		{
			stiffness: 0.2,
			damping: 0.4
		}
	)

	function checkOverflow() {
		let offset = 10
		let navHeight = 50

		// Calcola la posizione massima consentita per la finestra
		const maxX = width - size[0]
		const maxY = height - size[1] - navHeight

		// Controlla se la finestra è al di fuori dei bordi orizzontali
		let newX = $coords.x
		if ($coords.x < 0) {
			// Riporta la finestra all'inizio della finestra visibile
			newX = 0 + offset
		} else if ($coords.x > maxX) {
			// Riporta la finestra alla fine della finestra visibile
			newX = maxX - offset
		}

		// Controlla se la finestra è al di fuori dei bordi verticali
		let newY = $coords.y
		if ($coords.y < 0) {
			// Riporta la finestra all'inizio della finestra visibile
			newY = 0 + offset
		} else if ($coords.y > maxY) {
			// Riporta la finestra alla fine della finestra visibile
			newY = maxY - offset
		}

		// Imposta le nuove coordinate della finestra
		coords.set({ x: newX, y: newY })

		windows.update((windows) => {
			return windows.map((window) => {
				if (window.id === id) {
					return { ...window, x: newX, y: newY }
				}
				return window
			})
		})
	}

	function handlePanStart() {
		coords.stiffness = coords.damping = 1
	}

	function handlePanMove(event: { detail: { dx: number; dy: number } }) {
		coords.update(($coords) => ({
			x: $coords.x + event.detail.dx,
			y: $coords.y + event.detail.dy
		}))
	}

	function handlePanEnd() {
		coords.stiffness = 0.2
		coords.damping = 0.4
		checkOverflow()
	}

	onMount(() => {
		coords.set({ x: posX, y: posY }, { hard: true })
	})
</script>

<svelte:window bind:innerWidth={width} bind:innerHeight={height} />

<div
	in:fade={{ duration: 30 }}
	out:fade={{ duration: 30 }}
	class="box"
	style="--width: {size[0]}px; --height: {size[1]}px; transform: translate({$coords.x}px,{$coords.y}px)">
	<div
		class="head"
		use:pannable
		on:panstart={handlePanStart}
		on:panmove={handlePanMove}
		on:panend={handlePanEnd}>
		<span class="name">{name}</span>
	</div>
	<div class="content">
		<svelte:component this={content} />
	</div>
</div>

<style>
	.box {
		position: absolute;
		width: var(--width);
		height: var(--height);
		border-radius: 6px;
		background-color: #eeeeee;
		border: 1px solid #cccccc;
	}

	.head {
		height: 42px;
		border-radius: 6px 6px 0px 0px;
		border-bottom: 1px solid #cccccc;
		user-select: none;
		cursor: grab;
		display: flex;
		align-items: center;
		position: relative;
	}

	.head:active {
		cursor: grabbing;
	}

	.name {
		margin-left: 15px;
		font-size: 18px;
	}

	.content {
		padding: 15px;
		font-size: 16px;
	}
</style>