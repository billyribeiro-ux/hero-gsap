<script lang="ts">
	import { onMount } from 'svelte';
	import gsap from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	interface Props {
		leftText?: string;
		rightText?: string;
		revealTitle?: string;
		revealDescription?: string;
	}

	let {
		leftText = 'Cre',
		rightText = 'ate.',
		revealTitle = "What's Behind the Curtain",
		revealDescription = 'The most powerful reveals are the ones that make the audience lean forward.'
	}: Props = $props();

	let sectionEl: HTMLElement | undefined = $state();

	onMount(() => {
		gsap.registerPlugin(ScrollTrigger);
		if (!sectionEl) return;

		const ctx = gsap.context(() => {
			const tl = gsap.timeline({
				scrollTrigger: {
					trigger: sectionEl,
					start: 'top 60%',
					end: 'center center',
					scrub: 1
				}
			});

			tl.to('.sw-vline', { scaleY: 1, duration: 0.3 })
				.to('.sw-left', { xPercent: -100, duration: 1, ease: 'power3.inOut' }, 0.3)
				.to('.sw-right', { xPercent: 100, duration: 1, ease: 'power3.inOut' }, 0.3)
				.to('.sw-vline', { scaleY: 0, transformOrigin: 'bottom', duration: 0.3 }, 0.8)
				.to('.sw-reveal', { opacity: 1, duration: 0.5 }, 0.8);
		}, sectionEl);

		return () => ctx.revert();
	});
</script>

<section bind:this={sectionEl} class="hero-split">
	<div class="sw-vline"></div>

	<div class="sw-left">
		<span class="half-text half-text-dark">{leftText}</span>
	</div>
	<div class="sw-right">
		<span class="half-text half-text-light">{rightText}</span>
	</div>

	<div class="sw-reveal">
		<h2>{revealTitle}</h2>
		<p>{revealDescription}</p>
	</div>
</section>

<style>
	.hero-split {
		position: relative;
		height: 100vh;
		overflow: hidden;
		display: flex;
		align-items: center;
		justify-content: center;
		background: #111;
	}

	.sw-vline {
		position: absolute;
		top: 0;
		left: 50%;
		width: 2px;
		height: 100%;
		background: #ff3c00;
		transform: translateX(-50%) scaleY(0);
		transform-origin: top;
		z-index: 10;
	}

	.sw-left,
	.sw-right {
		position: absolute;
		top: 0;
		width: 50%;
		height: 100%;
		z-index: 5;
		display: flex;
		align-items: center;
	}

	.sw-left {
		left: 0;
		justify-content: flex-end;
		background: #e8e4df;
	}

	.sw-right {
		right: 0;
		justify-content: flex-start;
		background: #1a1a1a;
	}

	.half-text {
		font-family: 'Instrument Serif', serif;
		font-size: clamp(3rem, 8vw, 7rem);
	}

	.half-text-dark {
		color: #1a1a1a;
		padding-right: 5px;
		font-style: italic;
	}

	.half-text-light {
		color: #e8e4df;
		padding-left: 5px;
	}

	.sw-reveal {
		position: relative;
		z-index: 1;
		text-align: center;
		opacity: 0;
	}

	.sw-reveal h2 {
		font-family: 'Syne', sans-serif;
		font-size: clamp(2rem, 5vw, 4rem);
		font-weight: 800;
		color: #fff;
		margin-bottom: 20px;
	}

	.sw-reveal p {
		font-size: 1rem;
		color: rgba(255, 255, 255, 0.5);
		max-width: 500px;
		margin: 0 auto;
		line-height: 1.7;
		font-weight: 300;
	}
</style>