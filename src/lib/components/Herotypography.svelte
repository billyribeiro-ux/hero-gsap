<script lang="ts">
	import { onMount } from 'svelte';
	import gsap from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	interface WordConfig {
		text: string;
		variant?: 'default' | 'accent' | 'outline';
	}

	interface Props {
		words?: WordConfig[];
		tagline?: string;
	}

	let {
		words = [
			{ text: 'Move', variant: 'default' },
			{ text: 'Before', variant: 'accent' },
			{ text: 'The', variant: 'outline' },
			{ text: 'Move', variant: 'default' }
		],
		tagline = 'Precision in every pixel'
	}: Props = $props();

	let sectionEl: HTMLElement | undefined = $state();

	const directions = [
		{ x: -100, y: 50, rotation: -15 },
		{ x: 0, y: 120, rotation: 5 },
		{ x: 80, y: -60, rotation: 10 },
		{ x: -60, y: -80, rotation: -8 }
	];

	onMount(() => {
		gsap.registerPlugin(ScrollTrigger);
		if (!sectionEl) return;

		const ctx = gsap.context(() => {
			const wordEls = gsap.utils.toArray<HTMLElement>('.kt-word');
			const dotEls = gsap.utils.toArray<HTMLElement>('.kt-dot');

			const tl = gsap.timeline({
				scrollTrigger: {
					trigger: sectionEl,
					start: 'top 60%',
					toggleActions: 'play none none reverse'
				}
			});

			wordEls.forEach((word, i) => {
				const dir = directions[i % directions.length];
				gsap.set(word, { x: dir.x, y: dir.y, rotation: dir.rotation, opacity: 0 });

				tl.to(
					word,
					{
						x: 0,
						y: 0,
						rotation: 0,
						opacity: 1,
						duration: 1.2,
						ease: 'elastic.out(1, 0.5)'
					},
					0.15 * i
				);
			});

			tl.to('.kt-rule', { width: '200px', duration: 0.8, ease: 'power3.out' }, 0.6).to(
				'.kt-tagline',
				{ opacity: 1, duration: 0.6, ease: 'power2.out' },
				0.9
			);

			dotEls.forEach((dot, i) => {
				tl.to(dot, { opacity: 1, scale: 1, duration: 0.4 }, 0.3 + i * 0.1);

				gsap.to(dot, {
					y: 'random(-30, 30)',
					x: 'random(-20, 20)',
					duration: 'random(3, 6)',
					ease: 'sine.inOut',
					repeat: -1,
					yoyo: true,
					delay: Math.random() * 2
				});
			});
		}, sectionEl);

		return () => ctx.revert();
	});
</script>

<section bind:this={sectionEl} class="hero-kinetic">
	<div class="kinetic-container">
		{#each words as word}
			<span class="kt-word" class:accent={word.variant === 'accent'} class:outline={word.variant === 'outline'}>
				{word.text}
			</span>
		{/each}
		<div class="kt-rule"></div>
		<div class="kt-tagline">{tagline}</div>
	</div>

	<div class="kt-dot" style="top: 20%; left: 15%"></div>
	<div class="kt-dot" style="top: 70%; left: 80%"></div>
	<div class="kt-dot" style="top: 40%; right: 10%"></div>
	<div class="kt-dot" style="bottom: 20%; left: 30%"></div>
</section>

<style>
	.hero-kinetic {
		position: relative;
		height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		background: #faf8f4;
		overflow: hidden;
	}

	.kinetic-container {
		position: relative;
		z-index: 2;
		text-align: center;
		width: 80%;
		max-width: 900px;
	}

	:global(.kt-word) {
		display: inline-block;
		font-family: 'Playfair Display', serif;
		font-size: clamp(3rem, 7vw, 6rem);
		font-weight: 900;
		color: #1a1a1a;
		line-height: 1.1;
		margin: 0 15px;
		opacity: 0;
	}

	:global(.kt-word).accent {
		color: #ff3c00;
		font-style: italic;
	}

	:global(.kt-word).outline {
		-webkit-text-stroke: 2px #1a1a1a;
		color: transparent;
	}

	:global(.kt-rule) {
		width: 0;
		height: 1px;
		background: #1a1a1a;
		margin: 30px auto;
		opacity: 0.2;
	}

	:global(.kt-tagline) {
		font-family: 'Space Mono', monospace;
		font-size: 0.7rem;
		letter-spacing: 0.4em;
		text-transform: uppercase;
		color: #999;
		margin-top: 30px;
		opacity: 0;
	}

	:global(.kt-dot) {
		position: absolute;
		width: 8px;
		height: 8px;
		border-radius: 50%;
		background: #ff3c00;
		opacity: 0;
	}
</style>