<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { Waves, ArrowRight } from 'phosphor-svelte';

	let section: HTMLElement;
	let titleRef: HTMLElement;
	let subtitleRef: HTMLElement;
	let ctaRef: HTMLElement;
	let blob1: SVGPathElement;
	let blob2: SVGPathElement;
	let blob3: SVGPathElement;
	let linePathRef: SVGPathElement;
	let circlePath: SVGCircleElement;
	let counterRef: HTMLElement;
	let statsRef: HTMLElement;

	const morphShapes = [
		'M 400,200 C 550,80 720,100 780,280 C 840,460 740,620 560,650 C 380,680 200,580 160,400 C 120,220 250,320 400,200 Z',
		'M 420,160 C 600,60 760,160 800,340 C 840,520 700,660 500,660 C 300,660 140,520 160,340 C 180,160 240,260 420,160 Z',
		'M 380,220 C 520,60 740,80 800,300 C 860,520 720,660 520,640 C 320,620 160,500 180,320 C 200,140 240,380 380,220 Z',
		'M 440,180 C 600,80 760,180 790,360 C 820,540 680,660 480,650 C 280,640 140,500 170,320 C 200,140 280,280 440,180 Z',
	];

	const morphShapes2 = [
		'M 300,300 C 420,150 620,120 720,280 C 820,440 760,620 580,680 C 400,740 200,640 160,460 C 120,280 180,450 300,300 Z',
		'M 320,260 C 460,100 660,140 740,320 C 820,500 740,660 540,680 C 340,700 160,580 180,380 C 200,180 180,420 320,260 Z',
		'M 280,340 C 400,160 640,100 740,300 C 840,500 760,660 560,680 C 360,700 140,580 160,380 C 180,180 160,520 280,340 Z',
		'M 360,220 C 500,60 700,120 780,320 C 860,520 760,680 540,680 C 320,680 140,540 180,340 C 220,140 220,380 360,220 Z',
	];

	onMount(() => {
		let shapeIdx = 0;

		function morphBlobs() {
			shapeIdx = (shapeIdx + 1) % morphShapes.length;
			gsap.to(blob1, {
				attr: { d: morphShapes[shapeIdx] },
				duration: 4,
				ease: 'power2.inOut',
			});
			gsap.to(blob2, {
				attr: { d: morphShapes2[shapeIdx] },
				duration: 5,
				ease: 'power2.inOut',
				delay: 0.5,
			});
			gsap.to(blob3, {
				attr: { d: morphShapes[(shapeIdx + 2) % morphShapes.length] },
				duration: 6,
				ease: 'power2.inOut',
				delay: 1,
			});
		}

		const morphInterval = setInterval(morphBlobs, 3500);

		gsap.to(blob1, {
			rotate: 360,
			duration: 40,
			repeat: -1,
			ease: 'none',
			transformOrigin: '50% 50%',
		});
		gsap.to(blob2, {
			rotate: -360,
			duration: 55,
			repeat: -1,
			ease: 'none',
			transformOrigin: '50% 50%',
		});
		gsap.to(blob3, {
			rotate: 200,
			duration: 70,
			repeat: -1,
			ease: 'none',
			transformOrigin: '50% 50%',
		});

		const lineLen = linePathRef.getTotalLength();
		gsap.set(linePathRef, { strokeDasharray: lineLen, strokeDashoffset: lineLen });

		gsap.set([titleRef, subtitleRef, ctaRef, statsRef], { autoAlpha: 0, y: 40 });

		const tl = gsap.timeline({ delay: 0.2 });
		tl.to(linePathRef, { strokeDashoffset: 0, duration: 1.8, ease: 'power3.inOut' })
			.to(circlePath, { attr: { r: 6 }, duration: 0.4, ease: 'back.out(2)' }, '-=0.2')
			.to(titleRef, { autoAlpha: 1, y: 0, duration: 1, ease: 'power3.out' }, '-=0.3')
			.to(subtitleRef, { autoAlpha: 1, y: 0, duration: 0.9, ease: 'power3.out' }, '-=0.6')
			.to(statsRef, { autoAlpha: 1, y: 0, duration: 0.8, ease: 'power3.out' }, '-=0.5')
			.to(ctaRef, { autoAlpha: 1, y: 0, duration: 0.7, ease: 'power3.out' }, '-=0.4');

		let count = 0;
		gsap.to({ val: 0 }, {
			val: 100,
			duration: 2.5,
			ease: 'power2.out',
			delay: 1.2,
			onUpdate: function () {
				if (counterRef) counterRef.textContent = Math.round(this.targets()[0].val) + '%';
			},
		});

		const mousemove = (e: MouseEvent) => {
			const rect = section.getBoundingClientRect();
			const mx = (e.clientX - rect.left) / rect.width - 0.5;
			const my = (e.clientY - rect.top) / rect.height - 0.5;
			gsap.to('.h3__blobs', {
				x: mx * 40,
				y: my * 25,
				duration: 2,
				ease: 'power2.out',
			});
		};
		section.addEventListener('mousemove', mousemove);

		return () => {
			clearInterval(morphInterval);
			section.removeEventListener('mousemove', mousemove);
		};
	});
</script>

<section class="h3" bind:this={section}>
	<div class="h3__bg"></div>

	<svg class="h3__blobs" viewBox="0 0 960 960" preserveAspectRatio="xMidYMid slice" aria-hidden="true">
		<defs>
			<radialGradient id="blobGrad1" cx="50%" cy="50%" r="50%">
				<stop offset="0%" stop-color="#a855f7" stop-opacity="0.55"/>
				<stop offset="100%" stop-color="#6366f1" stop-opacity="0"/>
			</radialGradient>
			<radialGradient id="blobGrad2" cx="50%" cy="50%" r="50%">
				<stop offset="0%" stop-color="#06b6d4" stop-opacity="0.45"/>
				<stop offset="100%" stop-color="#3b82f6" stop-opacity="0"/>
			</radialGradient>
			<radialGradient id="blobGrad3" cx="50%" cy="50%" r="50%">
				<stop offset="0%" stop-color="#f472b6" stop-opacity="0.35"/>
				<stop offset="100%" stop-color="#a855f7" stop-opacity="0"/>
			</radialGradient>
			<filter id="blobBlur">
				<feGaussianBlur stdDeviation="22"/>
			</filter>
		</defs>
		<path
			bind:this={blob1}
			d={morphShapes[0]}
			fill="url(#blobGrad1)"
			filter="url(#blobBlur)"
		/>
		<path
			bind:this={blob2}
			d={morphShapes2[0]}
			fill="url(#blobGrad2)"
			filter="url(#blobBlur)"
		/>
		<path
			bind:this={blob3}
			d={morphShapes[2]}
			fill="url(#blobGrad3)"
			filter="url(#blobBlur)"
		/>
	</svg>

	<svg class="h3__line-art" viewBox="0 0 1200 600" aria-hidden="true">
		<path
			bind:this={linePathRef}
			d="M 0,500 Q 200,100 400,300 T 800,200 T 1200,400"
			fill="none"
			stroke="url(#lineGrad)"
			stroke-width="1.5"
		/>
		<circle
			bind:this={circlePath}
			cx="1200" cy="400" r="0"
			fill="#a855f7"
		/>
		<defs>
			<linearGradient id="lineGrad" x1="0%" y1="0%" x2="100%" y2="0%">
				<stop offset="0%" stop-color="#6366f1" stop-opacity="0"/>
				<stop offset="40%" stop-color="#a855f7" stop-opacity="0.8"/>
				<stop offset="100%" stop-color="#06b6d4" stop-opacity="0.9"/>
			</linearGradient>
		</defs>
	</svg>

	<div class="h3__content">
		<h1 class="h3__title" bind:this={titleRef}>
			<span class="h3__title-line">DESIGN</span>
			<span class="h3__title-line h3__title-outline">IN MOTION</span>
		</h1>

		<p class="h3__subtitle" bind:this={subtitleRef}>
			<Waves size={16} weight="fill" />
			&nbsp;Fluid interfaces that breathe, morph, and feel alive.
		</p>

		<div class="h3__stats" bind:this={statsRef}>
			<div class="h3__stat">
				<span class="h3__stat-num" bind:this={counterRef}>0%</span>
				<span class="h3__stat-label">Flow index</span>
			</div>
			<div class="h3__divider"></div>
			<div class="h3__stat">
				<span class="h3__stat-num">∞</span>
				<span class="h3__stat-label">Possibilities</span>
			</div>
			<div class="h3__divider"></div>
			<div class="h3__stat">
				<span class="h3__stat-num">60fps</span>
				<span class="h3__stat-label">Always</span>
			</div>
		</div>

		<div class="h3__cta" bind:this={ctaRef}>
			<button class="h3__btn">
				Explore the Flow
				<ArrowRight size={16} weight="bold" />
			</button>
		</div>
	</div>

	<div class="h3__noise"></div>
	<div class="h3__vignette"></div>
</section>

<style>
	.h3 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #06030f;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.h3__bg {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at 30% 40%, rgba(99, 60, 200, 0.12) 0%, transparent 60%),
			radial-gradient(ellipse at 70% 60%, rgba(6, 182, 212, 0.08) 0%, transparent 60%);
	}

	.h3__blobs {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		z-index: 1;
	}

	.h3__line-art {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		z-index: 2;
	}

	.h3__noise {
		position: absolute;
		inset: 0;
		background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.04'/%3E%3C/svg%3E");
		background-size: 200px 200px;
		pointer-events: none;
		z-index: 3;
		opacity: 0.4;
	}

	.h3__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at center, transparent 35%, rgba(6, 3, 15, 0.88) 100%);
		pointer-events: none;
		z-index: 4;
	}

	.h3__content {
		position: relative;
		z-index: 5;
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 2rem;
	}

	.h3__title {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.1em;
		line-height: 0.9;
	}

	.h3__title-line {
		font-family: var(--font-cinematic);
		font-size: clamp(5rem, 14vw, 13rem);
		letter-spacing: 0.08em;
		color: #fff;
		text-shadow: 0 0 60px rgba(168, 85, 247, 0.5), 0 0 120px rgba(168, 85, 247, 0.2);
	}

	.h3__title-outline {
		color: transparent;
		-webkit-text-stroke: 1.5px rgba(168, 85, 247, 0.8);
		text-shadow: none;
		filter: drop-shadow(0 0 20px rgba(168, 85, 247, 0.4));
	}

	.h3__subtitle {
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 0.9rem;
		color: rgba(200, 180, 240, 0.7);
		font-weight: 300;
		letter-spacing: 0.05em;
		gap: 0.3rem;
	}

	.h3__stats {
		display: flex;
		align-items: center;
		gap: 2rem;
	}

	.h3__stat {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.3rem;
	}

	.h3__stat-num {
		font-family: var(--font-display);
		font-size: 1.9rem;
		font-weight: 700;
		color: #fff;
		letter-spacing: -0.02em;
	}

	.h3__stat-label {
		font-size: 0.68rem;
		text-transform: uppercase;
		letter-spacing: 0.12em;
		color: rgba(160, 140, 200, 0.6);
	}

	.h3__divider {
		width: 1px;
		height: 36px;
		background: rgba(168, 85, 247, 0.25);
	}

	.h3__cta {
		margin-top: 0.5rem;
	}

	.h3__btn {
		display: inline-flex;
		align-items: center;
		gap: 0.6rem;
		padding: 0.9rem 2.2rem;
		background: linear-gradient(135deg, rgba(168, 85, 247, 0.2), rgba(99, 102, 241, 0.2));
		border: 1px solid rgba(168, 85, 247, 0.5);
		border-radius: 99px;
		color: rgba(220, 200, 255, 0.95);
		font-size: 0.88rem;
		font-weight: 600;
		cursor: pointer;
		backdrop-filter: blur(12px);
		transition: all 0.3s ease;
		box-shadow: 0 0 30px rgba(168, 85, 247, 0.2), inset 0 1px 0 rgba(255,255,255,0.1);
	}

	.h3__btn:hover {
		background: linear-gradient(135deg, rgba(168, 85, 247, 0.35), rgba(99, 102, 241, 0.35));
		box-shadow: 0 0 60px rgba(168, 85, 247, 0.45);
		transform: translateY(-2px);
	}
</style>
