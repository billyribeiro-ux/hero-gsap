<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { FilmSlate, Star, ArrowUpRight } from 'phosphor-svelte';

	let section: HTMLElement;
	let burnCanvas: HTMLCanvasElement;
	let titleLine1: HTMLElement;
	let titleLine2: HTMLElement;
	let titleLine3: HTMLElement;
	let subtitleRef: HTMLElement;
	let ctaRef: HTMLElement;
	let filmStrip: HTMLElement;
	let awardBadge: HTMLElement;
	let dividerRef: HTMLElement;

	onMount(() => {
		// Film burn canvas
		const c = burnCanvas.getContext('2d')!;
		let W = burnCanvas.width = window.innerWidth;
		let H = burnCanvas.height = window.innerHeight;
		let burnPhase = 0;
		let burnActive = true;
		let raf: number;

		function drawBurn() {
			if (!burnActive) return;
			c.clearRect(0, 0, W, H);
			burnPhase += 0.018;

			// Film burn organic edge
			const edgeCount = 6;
			for (let e = 0; e < edgeCount; e++) {
				const progress = Math.max(0, Math.min(1, (burnPhase - e * 0.15) * 1.4));
				if (progress <= 0) continue;

				const grad = c.createRadialGradient(
					W * (0.3 + Math.sin(burnPhase * 0.4 + e) * 0.3),
					H * (0.4 + Math.cos(burnPhase * 0.3 + e * 0.7) * 0.3),
					0,
					W * 0.5, H * 0.5,
					W * (0.5 + progress * 0.6)
				);
				const orange = `rgba(${200 + Math.random() * 55},${60 + Math.random() * 40},0,`;
				grad.addColorStop(0, orange + `${0.55 * (1 - progress)})`);
				grad.addColorStop(0.4, orange + `${0.3 * (1 - progress)})`);
				grad.addColorStop(0.7, `rgba(80,20,0,${0.15 * (1 - progress)})`);
				grad.addColorStop(1, 'rgba(0,0,0,0)');
				c.fillStyle = grad;
				c.fillRect(0, 0, W, H);
			}

			// Burn hole at center — punching through
			if (burnPhase > 0.5 && burnPhase < 2.5) {
				const holeSize = Math.min(W * 1.5, (burnPhase - 0.5) * W * 0.9);
				const holeGrad = c.createRadialGradient(W/2, H/2, 0, W/2, H/2, holeSize);
				holeGrad.addColorStop(0, 'rgba(0,0,0,0)');
				holeGrad.addColorStop(0.5, `rgba(180,60,0,${Math.max(0, 0.4 - (burnPhase - 0.5) * 0.2)})`);
				holeGrad.addColorStop(0.85, `rgba(255,120,0,${Math.max(0, 0.6 - (burnPhase - 0.5) * 0.3)})`);
				holeGrad.addColorStop(1, 'rgba(0,0,0,0)');
				c.fillStyle = holeGrad;
				c.fillRect(0, 0, W, H);
			}

			// Flicker noise grain
			if (burnPhase < 3) {
				const grainAlpha = Math.max(0, 0.06 - burnPhase * 0.02);
				for (let i = 0; i < 800; i++) {
					const gx = Math.random() * W;
					const gy = Math.random() * H;
					const gs = Math.random() * 3;
					c.fillStyle = `rgba(255,${100 + Math.random()*155},0,${grainAlpha * Math.random()})`;
					c.fillRect(gx, gy, gs, gs);
				}
			}

			if (burnPhase > 3.5) {
				burnActive = false;
				gsap.to(burnCanvas, { autoAlpha: 0, duration: 0.6 });
				return;
			}
			raf = requestAnimationFrame(drawBurn);
		}
		raf = requestAnimationFrame(drawBurn);

		// Film strip frame numbers
		const frameNums = filmStrip.querySelectorAll<HTMLElement>('.h6__frame-num');
		let frame = 1;
		const frameInterval = setInterval(() => {
			frameNums.forEach(el => { el.textContent = String(frame).padStart(4, '0'); });
			frame++;
		}, 42);

		// Entrance timeline
		gsap.set([titleLine1, titleLine2, titleLine3, subtitleRef, ctaRef, awardBadge, dividerRef], {
			autoAlpha: 0,
		});
		gsap.set(titleLine1, { y: 80, skewX: -3 });
		gsap.set(titleLine2, { y: 80, skewX: 3 });
		gsap.set(titleLine3, { y: 60, skewX: -2 });
		gsap.set(filmStrip, { x: '100%' });

		const tl = gsap.timeline({ delay: 0.8 });

		tl.to(filmStrip, { x: '0%', duration: 1.2, ease: 'expo.out' })
			.to(awardBadge, { autoAlpha: 1, duration: 0.6, ease: 'power2.out' }, '-=0.4')
			.to(titleLine1, { autoAlpha: 1, y: 0, skewX: 0, duration: 1.1, ease: 'expo.out' }, '-=0.2')
			.to(dividerRef, { autoAlpha: 1, scaleX: 1, duration: 0.8, ease: 'power3.out', transformOrigin: 'left center' }, '-=0.5')
			.to(titleLine2, { autoAlpha: 1, y: 0, skewX: 0, duration: 1.0, ease: 'expo.out' }, '-=0.6')
			.to(titleLine3, { autoAlpha: 1, y: 0, skewX: 0, duration: 0.9, ease: 'expo.out' }, '-=0.65')
			.to(subtitleRef, { autoAlpha: 1, duration: 0.8, ease: 'power2.out' }, '-=0.4')
			.to(ctaRef, { autoAlpha: 1, duration: 0.7, ease: 'power2.out' }, '-=0.4');

		// Ambient title glow pulse
		gsap.to([titleLine1, titleLine2], {
			textShadow: '0 0 100px rgba(255,200,80,0.5), 0 0 200px rgba(255,150,30,0.25)',
			duration: 3,
			repeat: -1,
			yoyo: true,
			ease: 'sine.inOut',
			delay: 3,
		});

		// Parallax on scroll simulation via mouse
		const onMouse = (e: MouseEvent) => {
			const rx = (e.clientY / window.innerHeight - 0.5) * 8;
			const ry = (e.clientX / window.innerWidth - 0.5) * -6;
			gsap.to('.h6__title-block', {
				rotateX: rx, rotateY: ry,
				duration: 1.6, ease: 'power2.out',
				transformPerspective: 900,
			});
			gsap.to(filmStrip, {
				y: (e.clientY / window.innerHeight - 0.5) * 14,
				duration: 1.8, ease: 'power2.out',
			});
		};
		section.addEventListener('mousemove', onMouse);

		const onResize = () => {
			W = burnCanvas.width = window.innerWidth;
			H = burnCanvas.height = window.innerHeight;
		};
		window.addEventListener('resize', onResize);

		return () => {
			cancelAnimationFrame(raf);
			clearInterval(frameInterval);
			section.removeEventListener('mousemove', onMouse);
			window.removeEventListener('resize', onResize);
		};
	});
</script>

<section class="h6" bind:this={section}>
	<canvas class="h6__burn" bind:this={burnCanvas}></canvas>

	<div class="h6__bg-texture"></div>
	<div class="h6__grain"></div>

	<!-- Film strip side bar -->
	<div class="h6__film-strip" bind:this={filmStrip}>
		{#each { length: 18 } as _, i}
			<div class="h6__film-frame">
				<span class="h6__frame-num">0001</span>
				<div class="h6__frame-inner">
					<div class="h6__frame-noise"></div>
				</div>
			</div>
		{/each}
	</div>

	<div class="h6__content">
		<div class="h6__award-badge" bind:this={awardBadge}>
			<FilmSlate size={14} weight="fill" />
			<span>BEST PICTURE · CANNES 2025</span>
			<Star size={11} weight="fill" />
		</div>

		<div class="h6__title-block">
			<div class="h6__title-line" bind:this={titleLine1}>
				<span class="h6__title-seq">I.</span>
				<span class="h6__title-word">THE</span>
			</div>

			<div class="h6__divider" bind:this={dividerRef}></div>

			<div class="h6__title-line h6__title-line--big" bind:this={titleLine2}>
				<span class="h6__title-word h6__title-main">LAST</span>
			</div>

			<div class="h6__title-line h6__title-line--accent" bind:this={titleLine3}>
				<span class="h6__title-word">FRAME</span>
			</div>
		</div>

		<p class="h6__subtitle" bind:this={subtitleRef}>
			A film about memory, loss, and the one second<br />
			before everything changes forever.
		</p>

		<div class="h6__cta" bind:this={ctaRef}>
			<button class="h6__btn-watch">
				<span class="h6__play-icon">▶</span>
				Watch Now
			</button>
			<button class="h6__btn-info">
				Full Credits
				<ArrowUpRight size={14} weight="bold" />
			</button>
			<div class="h6__rating">
				<span class="h6__rating-label">R</span>
				<span>2h 38m</span>
				<span>·</span>
				<span>Drama</span>
			</div>
		</div>
	</div>

	<div class="h6__letterbox-top"></div>
	<div class="h6__letterbox-bot"></div>
	<div class="h6__vignette"></div>
	<div class="h6__scanlines"></div>
</section>

<style>
	.h6 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #0a0800;
		display: flex;
		align-items: center;
		padding: 0 8vw;
	}

	.h6__burn {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		z-index: 20;
		pointer-events: none;
		mix-blend-mode: screen;
	}

	.h6__bg-texture {
		position: absolute;
		inset: 0;
		background:
			radial-gradient(ellipse at 20% 50%, rgba(80, 50, 10, 0.25) 0%, transparent 55%),
			radial-gradient(ellipse at 80% 50%, rgba(40, 20, 5, 0.2) 0%, transparent 55%),
			linear-gradient(180deg, #1a1008 0%, #0a0800 40%, #060400 100%);
	}

	.h6__grain {
		position: absolute;
		inset: 0;
		background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 512 512' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='g'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.75' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23g)' opacity='0.07'/%3E%3C/svg%3E");
		background-size: 256px 256px;
		pointer-events: none;
		z-index: 2;
		opacity: 0.55;
		animation: grainShift 0.08s steps(1) infinite;
	}

	@keyframes grainShift {
		0%   { background-position: 0 0; }
		25%  { background-position: -64px 32px; }
		50%  { background-position: 48px -48px; }
		75%  { background-position: -32px 64px; }
		100% { background-position: 64px -16px; }
	}

	.h6__film-strip {
		position: absolute;
		right: 0;
		top: 0;
		bottom: 0;
		width: 88px;
		background: #0d0b00;
		border-left: 2px solid rgba(255, 200, 60, 0.08);
		display: flex;
		flex-direction: column;
		overflow: hidden;
		z-index: 5;
	}

	.h6__film-frame {
		flex: 1;
		border-bottom: 1px solid rgba(255, 200, 60, 0.06);
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 4px;
		padding: 4px;
		position: relative;
	}

	.h6__film-frame::before,
	.h6__film-frame::after {
		content: '';
		position: absolute;
		width: 8px; height: 8px;
		border-radius: 2px;
		background: rgba(0,0,0,0.9);
		border: 1px solid rgba(255,200,60,0.15);
	}

	.h6__film-frame::before { top: 4px; left: 6px; }
	.h6__film-frame::after  { bottom: 4px; left: 6px; }

	.h6__frame-num {
		font-family: var(--font-mono);
		font-size: 0.42rem;
		color: rgba(255, 200, 60, 0.3);
		letter-spacing: 0.08em;
	}

	.h6__frame-inner {
		width: 60px;
		height: 40px;
		background: rgba(255, 200, 60, 0.04);
		border: 1px solid rgba(255, 200, 60, 0.08);
		border-radius: 2px;
		overflow: hidden;
	}

	.h6__frame-noise {
		width: 100%; height: 100%;
		background: repeating-linear-gradient(
			0deg,
			transparent,
			transparent 2px,
			rgba(255,200,60,0.02) 2px,
			rgba(255,200,60,0.02) 4px
		);
	}

	.h6__content {
		position: relative;
		z-index: 10;
		display: flex;
		flex-direction: column;
		gap: 2rem;
		max-width: 760px;
	}

	.h6__award-badge {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		font-family: var(--font-mono);
		font-size: 0.66rem;
		letter-spacing: 0.18em;
		color: rgba(255, 200, 80, 0.8);
		text-transform: uppercase;
		padding: 0.35rem 0.9rem;
		border: 1px solid rgba(255, 200, 60, 0.25);
		width: fit-content;
		background: rgba(255, 200, 60, 0.04);
	}

	.h6__title-block {
		display: flex;
		flex-direction: column;
		gap: 0.2rem;
		transform-style: preserve-3d;
	}

	.h6__title-line {
		display: flex;
		align-items: baseline;
		gap: 1rem;
		line-height: 1;
	}

	.h6__title-seq {
		font-family: var(--font-mono);
		font-size: 1rem;
		color: rgba(255, 200, 60, 0.5);
		letter-spacing: 0.1em;
	}

	.h6__title-word {
		font-family: var(--font-cinematic);
		letter-spacing: 0.07em;
		color: #f5edd8;
		text-shadow: 0 2px 40px rgba(0,0,0,0.8);
	}

	.h6__title-line .h6__title-word {
		font-size: clamp(3.5rem, 9vw, 9rem);
	}

	.h6__title-line--big .h6__title-word {
		font-size: clamp(6rem, 17vw, 17rem) !important;
		line-height: 0.85;
		text-shadow:
			0 0 40px rgba(255, 200, 60, 0.35),
			0 0 100px rgba(255, 160, 30, 0.15),
			0 4px 80px rgba(0,0,0,0.9);
	}

	.h6__title-main {
		background: linear-gradient(160deg, #fff8e8 0%, #ffd060 40%, #ff9820 70%, #c05010 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
	}

	.h6__title-line--accent .h6__title-word {
		font-size: clamp(4rem, 11vw, 11rem) !important;
		color: transparent !important;
		-webkit-text-stroke: 1.5px rgba(255, 200, 60, 0.55);
		text-shadow: none !important;
		letter-spacing: 0.18em;
	}

	.h6__divider {
		width: 120px;
		height: 1px;
		background: linear-gradient(90deg, rgba(255,200,60,0.7), transparent);
		margin: 0.4rem 0;
	}

	.h6__subtitle {
		font-size: 0.92rem;
		color: rgba(220, 200, 160, 0.6);
		font-weight: 300;
		line-height: 1.8;
		letter-spacing: 0.03em;
		max-width: 420px;
	}

	.h6__cta {
		display: flex;
		align-items: center;
		gap: 1.2rem;
		flex-wrap: wrap;
	}

	.h6__btn-watch {
		display: inline-flex;
		align-items: center;
		gap: 0.7rem;
		padding: 0.85rem 2rem;
		background: linear-gradient(135deg, rgba(255, 190, 50, 0.2), rgba(200, 100, 20, 0.2));
		border: 1px solid rgba(255, 190, 50, 0.45);
		color: rgba(255, 230, 140, 0.95);
		font-family: var(--font-sans);
		font-size: 0.88rem;
		font-weight: 600;
		letter-spacing: 0.06em;
		cursor: pointer;
		transition: all 0.3s;
		backdrop-filter: blur(8px);
		box-shadow: 0 0 30px rgba(255, 190, 50, 0.15);
	}

	.h6__btn-watch:hover {
		background: linear-gradient(135deg, rgba(255, 190, 50, 0.35), rgba(200, 100, 20, 0.35));
		box-shadow: 0 0 60px rgba(255, 190, 50, 0.3);
		transform: translateY(-2px);
	}

	.h6__play-icon {
		font-size: 0.65rem;
		opacity: 0.9;
	}

	.h6__btn-info {
		display: inline-flex;
		align-items: center;
		gap: 0.4rem;
		padding: 0.85rem 1.6rem;
		background: transparent;
		border: 1px solid rgba(255,255,255,0.1);
		color: rgba(220, 200, 160, 0.6);
		font-size: 0.85rem;
		cursor: pointer;
		transition: all 0.25s;
	}

	.h6__btn-info:hover {
		border-color: rgba(255,255,255,0.2);
		color: rgba(255,230,160,0.9);
	}

	.h6__rating {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		font-family: var(--font-mono);
		font-size: 0.72rem;
		color: rgba(200, 180, 120, 0.5);
		letter-spacing: 0.08em;
		padding-left: 0.5rem;
		border-left: 1px solid rgba(255,200,60,0.15);
	}

	.h6__rating-label {
		padding: 0.1rem 0.35rem;
		border: 1px solid rgba(200, 180, 120, 0.4);
		font-size: 0.62rem;
		font-weight: 700;
	}

	.h6__letterbox-top,
	.h6__letterbox-bot {
		position: absolute;
		left: 0; right: 0;
		height: 10vh;
		background: #000;
		z-index: 15;
		pointer-events: none;
	}
	.h6__letterbox-top { top: 0; }
	.h6__letterbox-bot { bottom: 0; }

	.h6__vignette {
		position: absolute;
		inset: 0;
		background:
			radial-gradient(ellipse at center, transparent 35%, rgba(0,0,0,0.85) 100%);
		pointer-events: none;
		z-index: 3;
	}

	.h6__scanlines {
		position: absolute;
		inset: 0;
		background: repeating-linear-gradient(
			0deg,
			transparent,
			transparent 2px,
			rgba(0,0,0,0.04) 2px,
			rgba(0,0,0,0.04) 4px
		);
		pointer-events: none;
		z-index: 4;
	}
</style>
