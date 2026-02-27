<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { Mountains, ArrowRight, Compass } from 'phosphor-svelte';

	let section: HTMLElement;
	let layer1: HTMLElement;
	let layer2: HTMLElement;
	let layer3: HTMLElement;
	let layer4: HTMLElement;
	let titleRef: HTMLElement;
	let subtitleRef: HTMLElement;
	let ctaRef: HTMLElement;
	let letterboxTop: HTMLElement;
	let letterboxBot: HTMLElement;
	let depthLines: HTMLElement;

	const words = ['HORIZON', 'FRONTIER', 'INFINITY', 'BEYOND'];
	let wordIdx = 0;

	onMount(() => {
		gsap.set(letterboxTop, { y: 0 });
		gsap.set(letterboxBot, { y: 0 });
		gsap.set([titleRef, subtitleRef, ctaRef], { autoAlpha: 0, y: 50 });
		gsap.set(depthLines, { autoAlpha: 0 });

		const tl = gsap.timeline({ delay: 0.1 });

		tl.to([letterboxTop, letterboxBot], {
			y: (i) => (i === 0 ? '-100%' : '100%'),
			duration: 1.4,
			ease: 'expo.inOut',
		})
			.to(depthLines, { autoAlpha: 1, duration: 0.6, ease: 'power2.out' }, '-=0.4')
			.to(titleRef, { autoAlpha: 1, y: 0, duration: 1.1, ease: 'power3.out' }, '-=0.6')
			.to(subtitleRef, { autoAlpha: 1, y: 0, duration: 0.9, ease: 'power3.out' }, '-=0.6')
			.to(ctaRef, { autoAlpha: 1, y: 0, duration: 0.8, ease: 'power3.out' }, '-=0.5');

		const wordInterval = setInterval(() => {
			wordIdx = (wordIdx + 1) % words.length;
			const span = titleRef?.querySelector('.h4__word') as HTMLElement;
			if (!span) return;
			gsap.to(span, { autoAlpha: 0, y: -20, duration: 0.3, ease: 'power2.in', onComplete: () => {
				span.textContent = words[wordIdx];
				gsap.fromTo(span, { autoAlpha: 0, y: 20 }, { autoAlpha: 1, y: 0, duration: 0.5, ease: 'power3.out' });
			}});
		}, 2600);

		const onMouseMove = (e: MouseEvent) => {
			const rect = section.getBoundingClientRect();
			const mx = (e.clientX - rect.left) / rect.width - 0.5;
			const my = (e.clientY - rect.top) / rect.height - 0.5;

			gsap.to(layer1, { x: mx * -8, y: my * -5, duration: 1.8, ease: 'power2.out' });
			gsap.to(layer2, { x: mx * -20, y: my * -12, duration: 1.8, ease: 'power2.out' });
			gsap.to(layer3, { x: mx * -36, y: my * -22, duration: 1.8, ease: 'power2.out' });
			gsap.to(layer4, { x: mx * -55, y: my * -34, duration: 1.8, ease: 'power2.out' });
			gsap.to(titleRef, {
				rotateX: my * -5,
				rotateY: mx * 6,
				duration: 1.4,
				ease: 'power2.out',
				transformPerspective: 1000,
			});
		};

		section.addEventListener('mousemove', onMouseMove);

		gsap.to('.h4__mountain-line', {
			strokeDashoffset: 0,
			duration: 3,
			ease: 'power2.inOut',
			delay: 1.2,
			stagger: 0.15,
		});

		return () => {
			clearInterval(wordInterval);
			section.removeEventListener('mousemove', onMouseMove);
		};
	});
</script>

<section class="h4" bind:this={section}>
	<div class="h4__letterbox h4__letterbox-top" bind:this={letterboxTop}></div>
	<div class="h4__letterbox h4__letterbox-bot" bind:this={letterboxBot}></div>

	<div class="h4__layer h4__layer-1" bind:this={layer1}>
		<div class="h4__stars"></div>
	</div>

	<div class="h4__layer h4__layer-2" bind:this={layer2}>
		<svg class="h4__mountain-svg h4__mountain-far" viewBox="0 0 1600 500" preserveAspectRatio="xMidYMax meet">
			<defs>
				<linearGradient id="mtnGrad1" x1="0%" y1="0%" x2="0%" y2="100%">
					<stop offset="0%" stop-color="#1a1040" stop-opacity="0.9"/>
					<stop offset="100%" stop-color="#06030f" stop-opacity="1"/>
				</linearGradient>
			</defs>
			<path class="h4__mountain-line" d="M-100,500 L200,160 L380,280 L560,80 L740,240 L920,40 L1100,220 L1280,100 L1460,260 L1700,500 Z"
				fill="url(#mtnGrad1)" stroke="rgba(80,60,160,0.4)" stroke-width="1.5"
				style="stroke-dasharray:4000;stroke-dashoffset:4000"/>
		</svg>
	</div>

	<div class="h4__layer h4__layer-3" bind:this={layer3}>
		<svg class="h4__mountain-svg h4__mountain-mid" viewBox="0 0 1600 500" preserveAspectRatio="xMidYMax meet">
			<defs>
				<linearGradient id="mtnGrad2" x1="0%" y1="0%" x2="0%" y2="100%">
					<stop offset="0%" stop-color="#0d0820" stop-opacity="0.95"/>
					<stop offset="100%" stop-color="#06030f" stop-opacity="1"/>
				</linearGradient>
			</defs>
			<path class="h4__mountain-line" d="M-100,500 L100,300 L280,380 L420,180 L580,300 L720,120 L900,280 L1060,160 L1220,320 L1380,200 L1540,340 L1700,500 Z"
				fill="url(#mtnGrad2)" stroke="rgba(60,120,200,0.3)" stroke-width="1"
				style="stroke-dasharray:5000;stroke-dashoffset:5000"/>
		</svg>
	</div>

	<div class="h4__layer h4__layer-4" bind:this={layer4}>
		<svg class="h4__mountain-svg h4__mountain-near" viewBox="0 0 1600 500" preserveAspectRatio="xMidYMax meet">
			<defs>
				<linearGradient id="mtnGrad3" x1="0%" y1="0%" x2="0%" y2="100%">
					<stop offset="0%" stop-color="#080512" stop-opacity="0.98"/>
					<stop offset="100%" stop-color="#030209" stop-opacity="1"/>
				</linearGradient>
			</defs>
			<path class="h4__mountain-line" d="M-100,500 L80,380 L200,420 L320,300 L440,360 L560,240 L680,340 L800,220 L920,320 L1040,260 L1160,380 L1300,280 L1420,400 L1600,460 L1700,500 Z"
				fill="url(#mtnGrad3)" stroke="rgba(100,80,180,0.5)" stroke-width="1.5"
				style="stroke-dasharray:5500;stroke-dashoffset:5500"/>
		</svg>
		<div class="h4__fog"></div>
	</div>

	<div class="h4__depth-lines" bind:this={depthLines}>
		{#each { length: 12 } as _, i}
			<div class="h4__depth-line" style="--i:{i}"></div>
		{/each}
	</div>

	<div class="h4__content">
		<div class="h4__label">
			<Mountains size={14} weight="fill" />
			<span>PANORAMIC · SERIES IV</span>
		</div>

		<h1 class="h4__title" bind:this={titleRef}>
			<span class="h4__word">{words[0]}</span>
			<br />
			<span class="h4__title-sub">AWAITS</span>
		</h1>

		<p class="h4__subtitle" bind:this={subtitleRef}>
			Every layer tells a story. Every depth reveals another world.
		</p>

		<div class="h4__cta" bind:this={ctaRef}>
			<button class="h4__btn-primary">
				<Compass size={17} weight="fill" />
				Chart the Course
			</button>
			<button class="h4__btn-ghost">
				View Gallery
				<ArrowRight size={15} weight="bold" />
			</button>
		</div>
	</div>

	<div class="h4__vignette"></div>
	<div class="h4__grad-bottom"></div>
</section>

<style>
	.h4 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #06030f;
		display: flex;
		align-items: center;
		justify-content: center;
		perspective: 1200px;
	}

	.h4__letterbox {
		position: absolute;
		left: 0; right: 0;
		height: 14vh;
		background: #000;
		z-index: 20;
	}
	.h4__letterbox-top { top: 0; }
	.h4__letterbox-bot { bottom: 0; }

	.h4__layer {
		position: absolute;
		inset: -60px;
		z-index: 1;
		will-change: transform;
	}

	.h4__layer-1 { z-index: 1; }
	.h4__layer-2 { z-index: 2; }
	.h4__layer-3 { z-index: 3; }
	.h4__layer-4 { z-index: 4; }

	.h4__stars {
		position: absolute;
		inset: 0;
		background:
			radial-gradient(1px 1px at 10% 15%, rgba(255,255,255,0.7) 0%, transparent 100%),
			radial-gradient(1px 1px at 25% 8%, rgba(255,255,255,0.5) 0%, transparent 100%),
			radial-gradient(1.5px 1.5px at 40% 20%, rgba(255,255,255,0.6) 0%, transparent 100%),
			radial-gradient(1px 1px at 55% 12%, rgba(255,255,255,0.4) 0%, transparent 100%),
			radial-gradient(1px 1px at 70% 18%, rgba(255,255,255,0.65) 0%, transparent 100%),
			radial-gradient(1px 1px at 82% 9%, rgba(255,255,255,0.5) 0%, transparent 100%),
			radial-gradient(1px 1px at 15% 35%, rgba(255,255,255,0.4) 0%, transparent 100%),
			radial-gradient(1px 1px at 32% 42%, rgba(255,255,255,0.35) 0%, transparent 100%),
			radial-gradient(1px 1px at 60% 30%, rgba(255,255,255,0.5) 0%, transparent 100%),
			radial-gradient(1px 1px at 75% 38%, rgba(255,255,255,0.4) 0%, transparent 100%),
			radial-gradient(1px 1px at 88% 28%, rgba(255,255,255,0.55) 0%, transparent 100%),
			radial-gradient(2px 2px at 5% 55%, rgba(180,160,255,0.6) 0%, transparent 100%),
			radial-gradient(1.5px 1.5px at 48% 48%, rgba(160,200,255,0.5) 0%, transparent 100%),
			radial-gradient(1px 1px at 92% 50%, rgba(255,200,180,0.5) 0%, transparent 100%);
		background-color: transparent;
	}

	.h4__mountain-svg {
		position: absolute;
		bottom: -60px;
		left: -60px;
		right: -60px;
		width: calc(100% + 120px);
		height: auto;
	}

	.h4__fog {
		position: absolute;
		bottom: 0;
		left: -60px;
		right: -60px;
		height: 220px;
		background: linear-gradient(to top, rgba(6, 3, 15, 0.9) 0%, rgba(40, 20, 80, 0.15) 50%, transparent 100%);
		filter: blur(2px);
	}

	.h4__depth-lines {
		position: absolute;
		inset: 0;
		z-index: 5;
		pointer-events: none;
	}

	.h4__depth-line {
		position: absolute;
		left: 0; right: 0;
		height: 1px;
		background: linear-gradient(90deg, transparent, rgba(120, 100, 255, calc(0.04 + var(--i) * 0.015)), transparent);
		bottom: calc(var(--i) * 8%);
		transform: perspective(400px) rotateX(60deg);
		transform-origin: bottom center;
	}

	.h4__content {
		position: relative;
		z-index: 10;
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1.5rem;
	}

	.h4__label {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		font-family: var(--font-mono);
		font-size: 0.68rem;
		letter-spacing: 0.18em;
		color: rgba(140, 120, 200, 0.8);
		text-transform: uppercase;
		border-bottom: 1px solid rgba(120, 100, 200, 0.3);
		padding-bottom: 0.5rem;
	}

	.h4__title {
		font-family: var(--font-cinematic);
		font-size: clamp(4.5rem, 13vw, 13rem);
		line-height: 0.9;
		letter-spacing: 0.06em;
		color: #fff;
		text-shadow:
			0 0 40px rgba(100, 80, 200, 0.5),
			0 0 120px rgba(60, 100, 200, 0.2),
			0 2px 60px rgba(0,0,0,0.8);
		transform-style: preserve-3d;
	}

	.h4__word {
		display: inline-block;
		background: linear-gradient(135deg, #fff 0%, #c4b5fd 60%, #93c5fd 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
	}

	.h4__title-sub {
		display: block;
		font-size: 0.55em;
		letter-spacing: 0.4em;
		color: rgba(180, 160, 240, 0.6);
		-webkit-text-fill-color: rgba(180, 160, 240, 0.6);
		text-shadow: none;
	}

	.h4__subtitle {
		font-size: 0.92rem;
		color: rgba(180, 170, 220, 0.6);
		font-weight: 300;
		letter-spacing: 0.06em;
		max-width: 420px;
	}

	.h4__cta {
		display: flex;
		gap: 0.9rem;
		margin-top: 0.5rem;
	}

	.h4__btn-primary {
		display: inline-flex;
		align-items: center;
		gap: 0.55rem;
		padding: 0.82rem 1.9rem;
		background: rgba(100, 80, 200, 0.25);
		border: 1px solid rgba(140, 120, 255, 0.5);
		border-radius: 6px;
		color: rgba(220, 210, 255, 0.95);
		font-size: 0.86rem;
		font-weight: 600;
		cursor: pointer;
		backdrop-filter: blur(10px);
		transition: all 0.25s;
		box-shadow: 0 0 30px rgba(100, 80, 200, 0.2);
	}

	.h4__btn-primary:hover {
		background: rgba(100, 80, 200, 0.4);
		box-shadow: 0 0 50px rgba(100, 80, 200, 0.4);
		transform: translateY(-2px);
	}

	.h4__btn-ghost {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0.82rem 1.9rem;
		background: transparent;
		border: 1px solid rgba(255, 255, 255, 0.12);
		border-radius: 6px;
		color: rgba(200, 195, 230, 0.7);
		font-size: 0.86rem;
		cursor: pointer;
		transition: all 0.25s;
	}

	.h4__btn-ghost:hover {
		border-color: rgba(255, 255, 255, 0.25);
		color: #fff;
	}

	.h4__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at center, transparent 30%, rgba(6, 3, 15, 0.8) 100%);
		pointer-events: none;
		z-index: 8;
	}

	.h4__grad-bottom {
		position: absolute;
		bottom: 0;
		left: 0; right: 0;
		height: 35%;
		background: linear-gradient(to top, rgba(6, 3, 15, 1) 0%, transparent 100%);
		pointer-events: none;
		z-index: 9;
	}
</style>
