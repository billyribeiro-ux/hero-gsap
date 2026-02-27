<script lang="ts">
	import { onMount } from 'svelte';
	import gsap from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	interface Props {
		text?: string;
		statusItems?: Array<{ label: string }>;
	}

	let {
		text = 'OVERRIDE',
		statusItems = [{ label: 'SYS.ONLINE' }, { label: 'LAT.0.02MS' }, { label: 'FPS.120' }]
	}: Props = $props();

	let sectionEl: HTMLElement | undefined = $state();
	let glitchEl: HTMLElement | undefined = $state();

	function runGlitchLoop(target: HTMLElement): void {
		const intensity = Math.random() * 20 + 5;

		const tl = gsap.timeline({
			onComplete: () => {
				gsap.delayedCall(Math.random() * 3 + 1, () => runGlitchLoop(target));
			}
		});

		tl.set(target, { skewX: Math.random() * 10 - 5 })
			.set(target, {
				textShadow: `${intensity}px 0 #ff0040, ${-intensity}px 0 #00ffea`
			})
			.set(
				target,
				{
					skewX: 0,
					textShadow: '3px 0 #ff0040, -3px 0 #00ffea'
				},
				'+=0.08'
			)
			.set(
				target,
				{
					textShadow: `${-intensity * 0.5}px ${intensity * 0.3}px #ff0040, ${intensity * 0.5}px ${-intensity * 0.3}px #00ffea`
				},
				'+=0.15'
			)
			.set(target, { textShadow: '2px 0 #ff0040, -2px 0 #00ffea' }, '+=0.06');
	}

	onMount(() => {
		gsap.registerPlugin(ScrollTrigger);
		if (!sectionEl || !glitchEl) return;

		const el = glitchEl!;

		const ctx = gsap.context(() => {
			const tl = gsap.timeline({
				scrollTrigger: {
					trigger: sectionEl,
					start: 'top 60%',
					toggleActions: 'play none none reverse'
				}
			});

			tl.to('.gl-corner', { opacity: 1, duration: 0.3, stagger: 0.1 })
				.fromTo(
					el,
					{ opacity: 0, scale: 0.8 },
					{ opacity: 1, scale: 1, duration: 0.1 },
					0.4
				)
				.to('.gl-status', { opacity: 1, duration: 0.4, stagger: 0.15 }, 0.8);

			gsap.set(el, { textShadow: '2px 0 #ff0040, -2px 0 #00ffea' });
			gsap.delayedCall(1, () => runGlitchLoop(el));

			// Noise movement
			gsap.to('.gl-noise', {
				backgroundPosition: '100px 100px',
				duration: 0.5,
				repeat: -1,
				ease: 'steps(4)'
			});
		}, sectionEl);

		return () => ctx.revert();
	});
</script>

<section bind:this={sectionEl} class="hero-glitch">
	<div class="gl-scanlines"></div>
	<div class="gl-noise"></div>

	<div bind:this={glitchEl} class="glitch-text" data-text={text}>
		{text}
	</div>

	<div class="gl-corner corner-tl"></div>
	<div class="gl-corner corner-tr"></div>
	<div class="gl-corner corner-bl"></div>
	<div class="gl-corner corner-br"></div>

	<div class="status-bar">
		{#each statusItems as item}
			<div class="gl-status">
				<span class="dot"></span>{item.label}
			</div>
		{/each}
	</div>
</section>

<style>
	.hero-glitch {
		position: relative;
		height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		background: #000;
		overflow: hidden;
	}

	.gl-scanlines {
		position: absolute;
		inset: 0;
		z-index: 3;
		pointer-events: none;
		background: repeating-linear-gradient(
			0deg,
			transparent,
			transparent 2px,
			rgba(0, 0, 0, 0.15) 2px,
			rgba(0, 0, 0, 0.15) 4px
		);
	}

	.gl-noise {
		position: absolute;
		inset: 0;
		opacity: 0.06;
		z-index: 4;
		pointer-events: none;
		background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)'/%3E%3C/svg%3E");
		background-size: 256px 256px;
	}

	.glitch-text {
		font-family: 'Anton', sans-serif;
		font-size: clamp(4rem, 14vw, 12rem);
		text-transform: uppercase;
		letter-spacing: 0.02em;
		position: relative;
		color: #fff;
		z-index: 2;
	}

	:global(.gl-corner) {
		position: absolute;
		width: 60px;
		height: 60px;
		border: 1px solid rgba(255, 255, 255, 0.1);
		opacity: 0;
	}

	.corner-tl {
		top: 40px;
		left: 40px;
		border-right: none;
		border-bottom: none;
	}
	.corner-tr {
		top: 40px;
		right: 40px;
		border-left: none;
		border-bottom: none;
	}
	.corner-bl {
		bottom: 40px;
		left: 40px;
		border-right: none;
		border-top: none;
	}
	.corner-br {
		bottom: 40px;
		right: 40px;
		border-left: none;
		border-top: none;
	}

	.status-bar {
		position: absolute;
		bottom: 40px;
		left: 50%;
		transform: translateX(-50%);
		display: flex;
		gap: 30px;
		z-index: 5;
	}

	:global(.gl-status) {
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.6rem;
		letter-spacing: 0.2em;
		color: rgba(255, 255, 255, 0.3);
		opacity: 0;
	}

	.dot {
		display: inline-block;
		width: 4px;
		height: 4px;
		border-radius: 50%;
		background: #00ff88;
		margin-right: 8px;
		vertical-align: middle;
	}
</style>