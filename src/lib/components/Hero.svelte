<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { Button, Tag } from 'carbon-components-svelte';
	import { ArrowRight, Sparkle, Lightning, ArrowUpRight, Star, Rocket } from 'phosphor-svelte';

	let heroRef: HTMLElement;
	let badgeRef: HTMLElement;
	let headlineRef: HTMLElement;
	let sublineRef: HTMLElement;
	let descRef: HTMLElement;
	let ctaRef: HTMLElement;
	let statsRef: HTMLElement;
	let glowRef: HTMLElement;
	let particlesRef: HTMLElement;

	const stats = [
		{ value: '99%', label: 'Uptime SLA' },
		{ value: '2.4ms', label: 'Avg latency' },
		{ value: '10M+', label: 'Requests/day' },
	];

	onMount(() => {
		const ctx = gsap.context(() => {
			const tl = gsap.timeline({ defaults: { ease: 'power3.out' } });

			gsap.set([badgeRef, headlineRef, sublineRef, descRef, ctaRef, statsRef], {
				autoAlpha: 0,
				y: 30,
			});

			gsap.set(glowRef, { autoAlpha: 0, scale: 0.6 });

			tl.to(glowRef, { autoAlpha: 1, scale: 1, duration: 1.4, ease: 'power2.out' })
				.to(badgeRef, { autoAlpha: 1, y: 0, duration: 0.6 }, '-=0.9')
				.to(headlineRef, { autoAlpha: 1, y: 0, duration: 0.75 }, '-=0.4')
				.to(sublineRef, { autoAlpha: 1, y: 0, duration: 0.7 }, '-=0.5')
				.to(descRef, { autoAlpha: 1, y: 0, duration: 0.65 }, '-=0.45')
				.to(ctaRef, { autoAlpha: 1, y: 0, duration: 0.6 }, '-=0.4')
				.to(statsRef, { autoAlpha: 1, y: 0, duration: 0.6 }, '-=0.35');

			gsap.to(glowRef, {
				scale: 1.08,
				duration: 4,
				repeat: -1,
				yoyo: true,
				ease: 'sine.inOut',
				delay: 1.5,
			});

			const particles = particlesRef?.querySelectorAll<HTMLElement>('.particle');
			particles?.forEach((p, i) => {
				gsap.to(p, {
					y: gsap.utils.random(-18, 18),
					x: gsap.utils.random(-10, 10),
					opacity: gsap.utils.random(0.3, 0.8),
					duration: gsap.utils.random(2.5, 5),
					repeat: -1,
					yoyo: true,
					ease: 'sine.inOut',
					delay: i * 0.3,
				});
			});
		}, heroRef);

		return () => ctx.revert();
	});
</script>

<section class="hero" bind:this={heroRef}>
	<div class="hero__glow" bind:this={glowRef}></div>
	<div class="hero__grid-overlay"></div>

	<div class="particles" bind:this={particlesRef} aria-hidden="true">
		{#each { length: 12 } as _, i}
			<span
				class="particle"
				style="left:{8 + i * 7.5}%; top:{10 + (i % 4) * 20}%; animation-delay:{i * 0.4}s"
			></span>
		{/each}
	</div>

	<div class="hero__content">
		<div class="hero__badge" bind:this={badgeRef}>
			<Sparkle size={14} weight="fill" />
			<span>Now in public beta</span>
			<Tag type="blue" size="sm">New</Tag>
		</div>

		<h1 class="hero__headline" bind:this={headlineRef}>
			Build faster.<br />
			<span class="hero__headline--accent">Ship with confidence.</span>
		</h1>

		<p class="hero__subline" bind:this={sublineRef}>
			The platform engineered for <strong>modern teams</strong>.
		</p>

		<p class="hero__desc" bind:this={descRef}>
			Streamline your workflow with real-time collaboration, instant deployments, and
			intelligent observability — all in one unified platform.
		</p>

		<div class="hero__cta" bind:this={ctaRef}>
			<Button kind="primary" icon={ArrowRight}>Get started free</Button>
			<Button kind="ghost" icon={ArrowUpRight}>
				Read the docs
			</Button>
		</div>

		<div class="hero__stats" bind:this={statsRef}>
			{#each stats as stat}
				<div class="hero__stat">
					<span class="hero__stat-value">{stat.value}</span>
					<span class="hero__stat-label">{stat.label}</span>
				</div>
			{/each}
		</div>
	</div>

	<div class="hero__visual">
		<div class="hero__card hero__card--main">
			<div class="hero__card-header">
				<Lightning size={18} weight="fill" class="icon-lightning" />
				<span>Live deployment</span>
				<span class="hero__card-status">
					<span class="pulse"></span>
					Active
				</span>
			</div>
			<div class="hero__card-metric">
				<span class="metric-value">2.4<span class="metric-unit">ms</span></span>
				<span class="metric-label">Response time</span>
			</div>
			<div class="hero__card-bar">
				<div class="bar-track">
					<div class="bar-fill" style="width: 84%"></div>
				</div>
				<span>84% efficiency</span>
			</div>
		</div>

		<div class="hero__card hero__card--secondary">
			<div class="hero__card-header">
				<Rocket size={16} weight="fill" />
				<span>Last deploy</span>
			</div>
			<code class="hero__card-code">v2.14.0 → production</code>
			<div class="hero__card-tags">
				<Tag type="green" size="sm">Success</Tag>
				<Tag type="cool-gray" size="sm">42s ago</Tag>
			</div>
		</div>

		<div class="hero__card hero__card--tertiary">
			<div class="hero__card-header">
				<Star size={15} weight="fill" />
				<span>Team activity</span>
			</div>
			<ul class="hero__activity">
				{#each ['Merged PR #241', 'Pipeline passed', 'Alert resolved'] as item}
					<li>
						<span class="dot"></span>
						{item}
					</li>
				{/each}
			</ul>
		</div>
	</div>
</section>

<style>
	.hero {
		position: relative;
		min-height: 100vh;
		display: grid;
		grid-template-columns: 1fr 1fr;
		align-items: center;
		gap: 4rem;
		padding: 6rem 6rem 4rem;
		overflow: hidden;
		background: var(--color-bg);
	}

	.hero__glow {
		position: absolute;
		top: -20%;
		left: -10%;
		width: 70%;
		height: 70%;
		background: radial-gradient(ellipse at center, var(--color-primary-glow) 0%, transparent 65%);
		pointer-events: none;
		z-index: 0;
	}

	.hero__grid-overlay {
		position: absolute;
		inset: 0;
		background-image:
			linear-gradient(var(--color-border) 1px, transparent 1px),
			linear-gradient(90deg, var(--color-border) 1px, transparent 1px);
		background-size: 48px 48px;
		mask-image: radial-gradient(ellipse 80% 80% at 50% 50%, black 30%, transparent 100%);
		pointer-events: none;
		z-index: 0;
	}

	.particles {
		position: absolute;
		inset: 0;
		pointer-events: none;
		z-index: 0;
	}

	.particle {
		position: absolute;
		width: 3px;
		height: 3px;
		border-radius: 50%;
		background: var(--color-primary-light);
		opacity: 0.25;
	}

	.hero__content {
		position: relative;
		z-index: 1;
		display: flex;
		flex-direction: column;
		gap: 1.5rem;
		max-width: 580px;
	}

	.hero__badge {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0.35rem 0.75rem 0.35rem 0.6rem;
		border: 1px solid var(--color-border);
		border-radius: 99px;
		background: rgba(99, 102, 241, 0.08);
		color: var(--color-primary-light);
		font-size: 0.78rem;
		font-weight: 500;
		width: fit-content;
		backdrop-filter: blur(8px);
	}

	.hero__headline {
		font-size: clamp(2.6rem, 5vw, 4rem);
		font-weight: 700;
		line-height: 1.12;
		letter-spacing: -0.03em;
		color: var(--color-text);
	}

	.hero__headline--accent {
		background: linear-gradient(135deg, var(--color-primary-light) 0%, var(--color-accent) 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
	}

	.hero__subline {
		font-size: 1.2rem;
		color: var(--color-text-muted);
		font-weight: 400;
	}

	.hero__subline strong {
		color: var(--color-text);
		font-weight: 600;
	}

	.hero__desc {
		font-size: 0.97rem;
		color: var(--color-text-muted);
		line-height: 1.7;
		max-width: 480px;
	}

	.hero__cta {
		display: flex;
		gap: 0.75rem;
		flex-wrap: wrap;
	}

	:global(.hero__cta .bx--btn--primary) {
		background-color: var(--color-primary);
		border-color: var(--color-primary);
	}

	:global(.hero__cta .bx--btn--primary:hover) {
		background-color: var(--color-primary-light);
		border-color: var(--color-primary-light);
	}

	:global(.hero__cta .bx--btn--ghost) {
		color: var(--color-text-muted);
		border-color: var(--color-border);
	}

	:global(.hero__cta .bx--btn--ghost:hover) {
		color: var(--color-text);
		background: rgba(255,255,255,0.05);
	}

	.hero__stats {
		display: flex;
		gap: 2rem;
		padding-top: 0.5rem;
		border-top: 1px solid var(--color-border);
	}

	.hero__stat {
		display: flex;
		flex-direction: column;
		gap: 0.2rem;
	}

	.hero__stat-value {
		font-size: 1.4rem;
		font-weight: 700;
		color: var(--color-text);
		font-variant-numeric: tabular-nums;
	}

	.hero__stat-label {
		font-size: 0.75rem;
		color: var(--color-text-muted);
		text-transform: uppercase;
		letter-spacing: 0.06em;
	}

	.hero__visual {
		position: relative;
		z-index: 1;
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}

	.hero__card {
		border: 1px solid var(--color-border);
		border-radius: 12px;
		background: rgba(26, 26, 46, 0.6);
		backdrop-filter: blur(12px);
		padding: 1.25rem 1.4rem;
		display: flex;
		flex-direction: column;
		gap: 0.85rem;
	}

	.hero__card--secondary,
	.hero__card--tertiary {
		display: grid;
		grid-template-columns: 1fr 1fr;
		align-items: start;
		gap: 0.7rem;
	}

	.hero__card-header {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		font-size: 0.82rem;
		color: var(--color-text-muted);
		font-weight: 500;
		grid-column: 1 / -1;
	}

	:global(.icon-lightning) {
		color: var(--color-accent);
	}

	.hero__card-status {
		margin-left: auto;
		display: flex;
		align-items: center;
		gap: 0.4rem;
		font-size: 0.75rem;
		color: #4ade80;
	}

	.pulse {
		width: 7px;
		height: 7px;
		border-radius: 50%;
		background: #4ade80;
		box-shadow: 0 0 0 0 rgba(74, 222, 128, 0.4);
		animation: pulse 2s infinite;
	}

	@keyframes pulse {
		0% { box-shadow: 0 0 0 0 rgba(74, 222, 128, 0.4); }
		70% { box-shadow: 0 0 0 8px rgba(74, 222, 128, 0); }
		100% { box-shadow: 0 0 0 0 rgba(74, 222, 128, 0); }
	}

	.metric-value {
		font-size: 2.4rem;
		font-weight: 700;
		color: var(--color-text);
		line-height: 1;
		font-variant-numeric: tabular-nums;
	}

	.metric-unit {
		font-size: 1rem;
		color: var(--color-text-muted);
		font-weight: 400;
		margin-left: 2px;
	}

	.metric-label {
		font-size: 0.75rem;
		color: var(--color-text-muted);
	}

	.bar-track {
		height: 5px;
		border-radius: 99px;
		background: rgba(255, 255, 255, 0.08);
		overflow: hidden;
		margin-bottom: 0.3rem;
	}

	.bar-fill {
		height: 100%;
		border-radius: 99px;
		background: linear-gradient(90deg, var(--color-primary), var(--color-accent));
	}

	.hero__card-bar {
		font-size: 0.72rem;
		color: var(--color-text-muted);
	}

	.hero__card-code {
		font-family: var(--font-mono);
		font-size: 0.78rem;
		color: var(--color-accent);
		background: rgba(6, 182, 212, 0.08);
		padding: 0.3rem 0.55rem;
		border-radius: 6px;
		white-space: nowrap;
	}

	.hero__card-tags {
		display: flex;
		gap: 0.4rem;
		align-items: center;
		flex-wrap: wrap;
	}

	.hero__activity {
		list-style: none;
		display: flex;
		flex-direction: column;
		gap: 0.45rem;
		grid-column: 1 / -1;
	}

	.hero__activity li {
		display: flex;
		align-items: center;
		gap: 0.55rem;
		font-size: 0.8rem;
		color: var(--color-text-muted);
	}

	.dot {
		width: 6px;
		height: 6px;
		border-radius: 50%;
		background: var(--color-primary-light);
		flex-shrink: 0;
	}

	@media (max-width: 1024px) {
		.hero {
			grid-template-columns: 1fr;
			padding: 5rem 2rem 3rem;
			gap: 3rem;
		}

		.hero__glow {
			left: -20%;
			width: 100%;
		}

		.hero__card--secondary,
		.hero__card--tertiary {
			grid-template-columns: 1fr;
		}
	}

	@media (max-width: 640px) {
		.hero {
			padding: 4rem 1.25rem 2.5rem;
		}

		.hero__stats {
			gap: 1.25rem;
		}
	}
</style>
