<script lang="ts">
	import { onMount } from 'svelte';
	import gsap from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	interface DataPoint {
		target: number;
		unit: string;
		label: string;
		barWidth: number;
	}

	interface Props {
		title?: string;
		data?: DataPoint[];
	}

	let {
		title = 'By the Numbers',
		data = [
			{ target: 18, unit: 'K+', label: 'Active Traders', barWidth: 92 },
			{ target: 573, unit: 'X', label: 'Best Return', barWidth: 100 },
			{ target: 99, unit: '.8%', label: 'Uptime', barWidth: 99 }
		]
	}: Props = $props();

	let sectionEl: HTMLElement | undefined = $state();
	let displayValues: number[] = $derived(data.map(() => 0));

	onMount(() => {
		gsap.registerPlugin(ScrollTrigger);
		if (!sectionEl) return;

		const ctx = gsap.context(() => {
			const tl = gsap.timeline({
				scrollTrigger: {
					trigger: sectionEl,
					start: 'top 60%',
					toggleActions: 'play none none reverse'
				}
			});

			tl.to('.dc-title', { opacity: 1, y: 0, duration: 0.8, ease: 'power3.out' })
				.to('.dc-divider', { scaleX: 1, duration: 0.6, ease: 'power3.out' }, 0.3)
				.to('.dc-cell', { opacity: 1, y: 0, duration: 0.8, stagger: 0.2, ease: 'power3.out' }, 0.4);

			// Counter animations
			data.forEach((point, i) => {
				const obj = { val: 0 };

				ScrollTrigger.create({
					trigger: sectionEl,
					start: 'top 60%',
					onEnter: () => {
						gsap.to(obj, {
							val: point.target,
							duration: 2,
							ease: 'power2.out',
							onUpdate: () => {
								displayValues[i] = Math.round(obj.val);
							}
						});
					}
				});
			});

			// Bar fills
			gsap.utils.toArray<HTMLElement>('.dc-bar-fill').forEach((bar) => {
				const width = bar.dataset.width;
				ScrollTrigger.create({
					trigger: sectionEl,
					start: 'top 50%',
					onEnter: () => {
						gsap.to(bar, {
							width: `${width}%`,
							duration: 1.5,
							delay: 0.8,
							ease: 'power3.out'
						});
					}
				});
			});
		}, sectionEl);

		return () => ctx.revert();
	});
</script>

<section bind:this={sectionEl} class="hero-data">
	<div class="radial-bg"></div>

	<div class="headline-top">
		<h2 class="dc-title">{title}</h2>
		<div class="dc-divider"></div>
	</div>

	<div class="data-grid">
		{#each data as point, i}
			<div class="dc-cell">
				<div class="number">
					<span class="counter">{displayValues[i]}</span><span class="unit">{point.unit}</span>
				</div>
				<div class="label">{point.label}</div>
				<div class="bar">
					<div class="dc-bar-fill" data-width={point.barWidth}></div>
				</div>
			</div>
		{/each}
	</div>
</section>

<style>
	.hero-data {
		position: relative;
		height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		background: #070b14;
		overflow: hidden;
	}

	.radial-bg {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at 50% 50%, rgba(0, 102, 255, 0.08) 0%, transparent 70%);
	}

	.headline-top {
		position: absolute;
		top: 15%;
		left: 50%;
		transform: translateX(-50%);
		text-align: center;
		z-index: 2;
	}

	:global(.dc-title) {
		font-family: 'Syne', sans-serif;
		font-size: clamp(1.5rem, 3vw, 2.5rem);
		font-weight: 800;
		color: #fff;
		opacity: 0;
		transform: translateY(20px);
	}

	:global(.dc-divider) {
		width: 40px;
		height: 2px;
		background: #0066ff;
		margin: 15px auto 0;
		transform: scaleX(0);
	}

	.data-grid {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		gap: 60px;
		position: relative;
		z-index: 2;
		text-align: center;
	}

	:global(.dc-cell) {
		opacity: 0;
		transform: translateY(40px);
	}

	.number {
		font-family: 'Bebas Neue', sans-serif;
		font-size: clamp(3rem, 8vw, 6rem);
		color: #fff;
		line-height: 1;
	}

	.unit {
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.7rem;
		letter-spacing: 0.3em;
		color: #0066ff;
	}

	.label {
		font-size: 0.8rem;
		color: rgba(255, 255, 255, 0.3);
		margin-top: 10px;
		font-weight: 300;
	}

	.bar {
		width: 100%;
		height: 2px;
		background: rgba(255, 255, 255, 0.06);
		margin-top: 20px;
		position: relative;
		overflow: hidden;
		border-radius: 1px;
	}

	:global(.dc-bar-fill) {
		position: absolute;
		top: 0;
		left: 0;
		height: 100%;
		background: linear-gradient(90deg, #0066ff, #00ffd5);
		width: 0;
		border-radius: 1px;
	}
</style>