<script lang="ts">
	import { onMount } from 'svelte';
	import gsap from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	interface CardData {
		icon: string;
		title: string;
		description: string;
		color: string;
	}

	interface Props {
		cards?: CardData[];
		sideTitle?: string;
		sideDescription?: string;
	}

	let {
		cards = [
			{
				icon: '⚡',
				title: 'Lightning Fast',
				description: 'Sub-millisecond execution for real-time trading decisions',
				color: 'rgba(139, 0, 255, 0.08)'
			},
			{
				icon: '🎯',
				title: 'Precision Signals',
				description: 'Institutional-grade alerts with confirmed momentum triggers',
				color: 'rgba(0, 102, 255, 0.08)'
			},
			{
				icon: '🔥',
				title: 'Explosive Setups',
				description: 'Identify the move before the move with proprietary analysis',
				color: 'rgba(255, 60, 0, 0.08)'
			}
		],
		sideTitle = 'Stack the odds in your favor',
		sideDescription = 'Three layers of confirmation. One decisive moment of action.'
	}: Props = $props();

	let sectionEl: HTMLElement | undefined = $state();

	onMount(() => {
		gsap.registerPlugin(ScrollTrigger);
		if (!sectionEl) return;

		const ctx = gsap.context(() => {
			const cardEls = gsap.utils.toArray<HTMLElement>('.cs-card');

			const tl = gsap.timeline({
				scrollTrigger: {
					trigger: sectionEl,
					start: 'top 60%',
					toggleActions: 'play none none reverse'
				}
			});

			cardEls.forEach((card, i) => {
				const offset = (i - 1) * 30;
				const rotate = (i - 1) * -4;
				const scale = 1 - (cardEls.length - 1 - i) * 0.05;

				tl.fromTo(
					card,
					{ y: 100, opacity: 0, rotation: 0, scale: 0.9 },
					{
						y: offset,
						opacity: 1,
						rotation: rotate,
						scale,
						duration: 0.8,
						ease: 'power3.out'
					},
					0.15 * i
				);
			});

			tl.to('.cs-side-title', { opacity: 1, x: 0, duration: 1, ease: 'power3.out' }, 0.5).to(
				'.cs-side-desc',
				{ opacity: 1, x: 0, duration: 1, ease: 'power3.out' },
				0.7
			);

			// Mouse depth effect
			const handleMouse = (e: MouseEvent) => {
				if (!sectionEl) return;
				const rect = sectionEl.getBoundingClientRect();
				const x = (e.clientX - rect.left) / rect.width - 0.5;
				const y = (e.clientY - rect.top) / rect.height - 0.5;

				cardEls.forEach((card, i) => {
					const depth = (i + 1) * 8;
					gsap.to(card, {
						x: x * depth,
						rotateY: x * 5,
						rotateX: -y * 3,
						duration: 0.8,
						ease: 'power2.out'
					});
				});
			};

			sectionEl!.addEventListener('mousemove', handleMouse);
		}, sectionEl);

		return () => ctx.revert();
	});
</script>

<section bind:this={sectionEl} class="hero-cards">
	<div class="card-stack">
		{#each cards as card, i}
			<div class="cs-card" style="background: {card.color}">
				<div class="card-icon">{card.icon}</div>
				<div class="card-title">{card.title}</div>
				<div class="card-desc">{card.description}</div>
			</div>
		{/each}
	</div>

	<div class="side-text">
		<h2 class="cs-side-title">{@html sideTitle.replace(/\n/g, '<br />')}</h2>
		<p class="cs-side-desc">{sideDescription}</p>
	</div>
</section>

<style>
	.hero-cards {
		position: relative;
		height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		background: linear-gradient(180deg, #0f0f0f 0%, #1a1118 100%);
		overflow: hidden;
	}

	.card-stack {
		position: relative;
		width: clamp(300px, 50vw, 500px);
		height: clamp(350px, 50vh, 450px);
	}

	:global(.cs-card) {
		position: absolute;
		inset: 0;
		border-radius: 20px;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 40px;
		backdrop-filter: blur(40px);
		border: 1px solid rgba(255, 255, 255, 0.08);
		opacity: 0;
	}

	.card-icon {
		width: 48px;
		height: 48px;
		border-radius: 12px;
		border: 1px solid rgba(255, 255, 255, 0.15);
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 1.2rem;
		margin-bottom: 24px;
	}

	.card-title {
		font-family: 'Syne', sans-serif;
		font-size: 1.3rem;
		font-weight: 700;
		color: #fff;
		margin-bottom: 12px;
	}

	.card-desc {
		font-size: 0.85rem;
		color: rgba(255, 255, 255, 0.4);
		text-align: center;
		line-height: 1.6;
		font-weight: 300;
	}

	.side-text {
		position: absolute;
		top: 50%;
		right: 8%;
		transform: translateY(-50%);
		z-index: 2;
	}

	:global(.cs-side-title) {
		font-family: 'Instrument Serif', serif;
		font-size: clamp(2rem, 4vw, 3.5rem);
		color: #fff;
		line-height: 1.2;
		font-style: italic;
		opacity: 0;
		transform: translateX(40px);
	}

	:global(.cs-side-desc) {
		font-size: 0.9rem;
		color: rgba(255, 255, 255, 0.35);
		max-width: 300px;
		margin-top: 15px;
		line-height: 1.7;
		font-weight: 300;
		opacity: 0;
		transform: translateX(40px);
	}
</style>