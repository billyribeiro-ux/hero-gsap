<script lang="ts">
	import { onMount } from 'svelte';
	import gsap from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	interface Props {
		words?: string[];
		badges?: Array<{ label: string; icon: string; position: 'tl' | 'tr' | 'bl' | 'br' }>;
	}

	let {
		words = ['Design', 'Beyond', 'Boundaries'],
		badges = [
			{ label: 'INTERACTIVE', icon: '✦', position: 'tl' },
			{ label: '3D DEPTH', icon: '◈', position: 'br' },
			{ label: 'PARALLAX', icon: '⬡', position: 'tr' }
		]
	}: Props = $props();

	let sectionEl: HTMLElement | undefined = $state();

	const badgePositions: Record<string, string> = {
		tl: 'top: 25%; left: 8%;',
		tr: 'top: 15%; right: 20%;',
		bl: 'bottom: 30%; left: 15%;',
		br: 'bottom: 30%; right: 10%;'
	};

	onMount(() => {
		gsap.registerPlugin(ScrollTrigger);
		if (!sectionEl) return;

		const ctx = gsap.context(() => {
			const tl = gsap.timeline({
				scrollTrigger: {
					trigger: sectionEl,
					start: 'top 80%',
					end: 'top 20%',
					toggleActions: 'play none none reverse'
				}
			});

			tl.to('.mp-orb', {
				opacity: 0.4,
				duration: 2,
				stagger: 0.3,
				ease: 'power2.out'
			})
				.to('.mp-word', {
					opacity: 1,
					y: 0,
					rotateX: 0,
					duration: 1.2,
					stagger: 0.15,
					ease: 'power4.out'
				}, 0.3)
				.to('.mp-badge', {
					opacity: 1,
					duration: 0.8,
					stagger: 0.2,
					ease: 'power2.out'
				}, 0.8);

			// Ambient orb float
			gsap.to('.mp-orb-1', { x: 30, y: -20, duration: 6, ease: 'sine.inOut', repeat: -1, yoyo: true });
			gsap.to('.mp-orb-2', { x: -25, y: 30, duration: 8, ease: 'sine.inOut', repeat: -1, yoyo: true });
			gsap.to('.mp-orb-3', { scale: 1.2, duration: 5, ease: 'sine.inOut', repeat: -1, yoyo: true });
		}, sectionEl);

		const handleMouse = (e: MouseEvent) => {
			if (!sectionEl) return;
			const rect = sectionEl.getBoundingClientRect();
			const x = (e.clientX - rect.left) / rect.width - 0.5;
			const y = (e.clientY - rect.top) / rect.height - 0.5;

			gsap.to('.mp-layer-bg', {
				x: x * 40,
				y: y * 40,
				rotateY: x * 5,
				rotateX: -y * 5,
				duration: 1.2,
				ease: 'power2.out'
			});
			gsap.to('.mp-layer-text', {
				x: x * -20,
				y: y * -20,
				duration: 1.2,
				ease: 'power2.out'
			});

			document.querySelectorAll('.mp-badge').forEach((badge, i) => {
				const multiplier = (i + 1) * 20;
				gsap.to(badge, {
					x: x * multiplier * (i % 2 === 0 ? 1 : -1),
					y: y * multiplier * 0.6,
					duration: 1.5,
					ease: 'power2.out'
				});
			});
		};

		sectionEl.addEventListener('mousemove', handleMouse);

		return () => {
			ctx.revert();
			sectionEl?.removeEventListener('mousemove', handleMouse);
		};
	});
</script>

<section bind:this={sectionEl} class="hero-magnetic">
	<div class="mp-layer-bg">
		<div class="mp-orb mp-orb-1"></div>
		<div class="mp-orb mp-orb-2"></div>
		<div class="mp-orb mp-orb-3"></div>
	</div>

	<div class="mp-layer-text">
		<div class="big-text">
			{#each words as word, i}
				<span class="mp-word">{word}</span>
				{#if i === 1}<br />{/if}
			{/each}
		</div>
	</div>

	{#each badges as badge}
		<div class="mp-badge" style={badgePositions[badge.position]}>
			{badge.icon} {badge.label}
		</div>
	{/each}
</section>

<style>
	.hero-magnetic {
		position: relative;
		height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		background: #05080f;
		overflow: hidden;
		perspective: 1200px;
	}

	.mp-layer-bg,
	.mp-layer-text {
		position: absolute;
		inset: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		will-change: transform;
	}

	:global(.mp-orb) {
		position: absolute;
		border-radius: 50%;
		filter: blur(80px);
		opacity: 0;
	}

	.mp-orb-1 {
		width: 500px;
		height: 500px;
		background: #0066ff;
		top: 20%;
		left: 15%;
	}

	.mp-orb-2 {
		width: 400px;
		height: 400px;
		background: #8b00ff;
		bottom: 20%;
		right: 15%;
	}

	.mp-orb-3 {
		width: 300px;
		height: 300px;
		background: #00ffd5;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
	}

	.big-text {
		font-family: 'Syne', sans-serif;
		font-size: clamp(3rem, 9vw, 8rem);
		font-weight: 800;
		letter-spacing: -0.04em;
		line-height: 1;
		text-align: center;
		color: #fff;
		position: relative;
		z-index: 5;
	}

	:global(.mp-word) {
		display: inline-block;
		opacity: 0;
		transform: translateY(60px) rotateX(40deg);
		transform-origin: bottom center;
	}

	:global(.mp-badge) {
		position: absolute;
		padding: 12px 24px;
		border: 1px solid rgba(255, 255, 255, 0.1);
		border-radius: 100px;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.65rem;
		letter-spacing: 0.15em;
		color: rgba(255, 255, 255, 0.5);
		backdrop-filter: blur(20px);
		background: rgba(255, 255, 255, 0.03);
		opacity: 0;
		z-index: 10;
	}
</style>