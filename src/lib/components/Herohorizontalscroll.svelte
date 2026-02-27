<script lang="ts">
	import { onMount } from 'svelte';
	import gsap from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	interface Panel {
		text: string;
		background: string;
	}

	interface Props {
		panels?: Panel[];
	}

	let {
		panels = [
			{ text: 'The scroll becomes a timeline', background: '#0d0d0d' },
			{ text: 'Every pixel is a frame in your story', background: '#0a0f1a' },
			{ text: 'Control the narrative with motion', background: '#1a0a0a' }
		]
	}: Props = $props();

	let wrapperEl: HTMLElement | undefined = $state();
	let trackEl: HTMLElement | undefined = $state();

	onMount(() => {
		gsap.registerPlugin(ScrollTrigger);
		if (!wrapperEl || !trackEl) return;

		const track = trackEl;
		const wrapper = wrapperEl;

		const ctx = gsap.context(() => {
			// Main horizontal scroll
			gsap.to(track, {
				x: () => -(track.scrollWidth - window.innerWidth),
				ease: 'none',
				scrollTrigger: {
					trigger: wrapper,
					start: 'top top',
					end: () => `+=${track.scrollWidth - window.innerWidth}`,
					scrub: 1,
					pin: true,
					anticipatePin: 1,
					invalidateOnRefresh: true
				}
			});

			// Panel text reveals
			const panelEls = gsap.utils.toArray<HTMLElement>('.hs-panel');
			panelEls.forEach((panel, i) => {
				const textEl = panel.querySelector('.hs-panel-text');
				if (!textEl) return;

				ScrollTrigger.create({
					trigger: wrapperEl,
					start: () =>
						`top+=${(i * (trackEl!.scrollWidth - window.innerWidth)) / panelEls.length} top`,
					onEnter: () =>
						gsap.to(textEl, { opacity: 1, x: 0, duration: 0.8, ease: 'power3.out' }),
					onLeaveBack: () => gsap.to(textEl, { opacity: 0, x: 80, duration: 0.4 })
				});
			});

			// Progress bar
			gsap.to('.hs-progress', {
				width: '100%',
				ease: 'none',
				scrollTrigger: {
					trigger: wrapperEl,
					start: 'top top',
					end: 'bottom bottom',
					scrub: true
				}
			});
		}, wrapperEl);

		return () => ctx.revert();
	});
</script>

<div bind:this={wrapperEl} class="hs-wrapper" style="height: {panels.length * 100}vh">
	<div bind:this={trackEl} class="hs-track" style="width: {panels.length * 100}vw">
		{#each panels as panel, i}
			<div class="hs-panel" style="background: {panel.background}">
				<div class="hs-panel-text">{panel.text}</div>
				<div class="hs-panel-num">{String(i + 1).padStart(2, '0')}</div>
			</div>
		{/each}
		<div class="hs-progress"></div>
	</div>
</div>

<style>
	.hs-wrapper {
		position: relative;
	}

	.hs-track {
		height: 100vh;
		display: flex;
		align-items: stretch;
		overflow: hidden;
		position: relative;
	}

	.hs-panel {
		width: 100vw;
		height: 100vh;
		flex-shrink: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
	}

	:global(.hs-panel-text) {
		font-family: 'Playfair Display', serif;
		font-size: clamp(2.5rem, 6vw, 5rem);
		font-weight: 900;
		color: #fff;
		text-align: center;
		max-width: 600px;
		line-height: 1.15;
		opacity: 0;
		transform: translateX(80px);
	}

	.hs-panel-num {
		position: absolute;
		bottom: 40px;
		left: 40px;
		font-family: 'Bebas Neue', sans-serif;
		font-size: 8rem;
		color: rgba(255, 255, 255, 0.04);
		line-height: 1;
	}

	:global(.hs-progress) {
		position: absolute;
		bottom: 0;
		left: 0;
		height: 3px;
		background: linear-gradient(90deg, #ff3c00, #ff9500);
		width: 0;
		z-index: 10;
	}
</style>