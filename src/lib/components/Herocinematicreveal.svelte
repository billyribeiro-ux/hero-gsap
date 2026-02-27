<script lang="ts">
	import { onMount } from 'svelte';
	import gsap from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	interface Props {
		headline?: string;
		eyebrow?: string;
		description?: string;
	}

	let { headline = 'EXPLOSIVE', eyebrow = 'Introducing the Future', description = 'Motion design that commands attention.' }: Props = $props();

	let sectionEl: HTMLElement | undefined = $state();
	let headlineEl: HTMLElement | undefined = $state();
	let chars: HTMLSpanElement[] = $state([]);

	const splitChars = $derived(headline.split(''));

	onMount(() => {
		gsap.registerPlugin(ScrollTrigger);

		const ctx = gsap.context(() => {
			const tl = gsap.timeline({ delay: 0.3 });

			tl.to('.h1-accent-line', {
				width: '80%',
				duration: 1.2,
				ease: 'power4.inOut'
			})
				.to('.h1-eyebrow', {
					opacity: 1,
					duration: 0.8,
					ease: 'power2.out'
				}, 0.3)
				.to('.h1-char', {
					y: 0,
					duration: 1.4,
					stagger: 0.04,
					ease: 'power4.out'
				}, 0.5)
				.to('.h1-accent-line', {
					width: '0%',
					duration: 0.8,
					ease: 'power4.inOut'
				}, 1.2)
				.to('.h1-sub', {
					opacity: 1,
					duration: 1,
					ease: 'power2.out'
				}, 1.5)
				.to('.h1-scroll-indicator', {
					opacity: 1,
					duration: 0.6,
					ease: 'power2.out'
				}, 2);
		}, sectionEl);

		return () => ctx.revert();
	});
</script>

<section bind:this={sectionEl} class="hero-cinematic">
	<div class="bg-grid"></div>
	<div class="h1-accent-line"></div>

	<div class="content">
		<div class="h1-eyebrow">{eyebrow}</div>
		<h1 bind:this={headlineEl} class="headline">
			<span class="line">
				{#each splitChars as char, i}
					<span class="h1-char" bind:this={chars[i]}>{char}</span>
				{/each}
			</span>
		</h1>
		<p class="h1-sub">{description}</p>
	</div>

	<div class="h1-scroll-indicator">
		<span>Scroll</span>
		<div class="line-down"></div>
	</div>
</section>

<style>
	.hero-cinematic {
		position: relative;
		height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		background: #0a0a0a;
		overflow: hidden;
	}

	.bg-grid {
		position: absolute;
		inset: 0;
		background-image: linear-gradient(rgba(255, 255, 255, 0.03) 1px, transparent 1px),
			linear-gradient(90deg, rgba(255, 255, 255, 0.03) 1px, transparent 1px);
		background-size: 60px 60px;
	}

	.h1-accent-line {
		position: absolute;
		height: 1px;
		background: linear-gradient(90deg, transparent, #ff3c00, transparent);
		width: 0;
		left: 50%;
		transform: translateX(-50%);
		top: 50%;
	}

	.content {
		position: relative;
		text-align: center;
		z-index: 2;
	}

	:global(.h1-eyebrow) {
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.65rem;
		letter-spacing: 0.5em;
		text-transform: uppercase;
		color: #ff3c00;
		margin-bottom: 30px;
		opacity: 0;
	}

	.headline {
		font-family: 'Bebas Neue', sans-serif;
		font-size: clamp(4rem, 12vw, 11rem);
		line-height: 0.9;
		letter-spacing: -0.02em;
		color: #fff;
	}

	.line {
		overflow: hidden;
		display: block;
	}

	:global(.h1-char) {
		display: inline-block;
		transform: translateY(120%);
		will-change: transform;
	}

	:global(.h1-sub) {
		font-size: 1rem;
		color: rgba(255, 255, 255, 0.4);
		margin-top: 40px;
		max-width: 400px;
		margin-inline: auto;
		opacity: 0;
		font-weight: 300;
		line-height: 1.6;
	}

	:global(.h1-scroll-indicator) {
		position: absolute;
		bottom: 40px;
		left: 50%;
		transform: translateX(-50%);
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 8px;
		opacity: 0;
	}

	:global(.h1-scroll-indicator) span {
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.55rem;
		letter-spacing: 0.3em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.3);
	}

	.line-down {
		width: 1px;
		height: 40px;
		background: rgba(255, 255, 255, 0.15);
	}
</style>