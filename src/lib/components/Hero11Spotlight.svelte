<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { Star, Trophy, ArrowRight } from 'phosphor-svelte';

	let section: HTMLElement;
	let canvas: HTMLCanvasElement;
	let curtainLeft: HTMLElement;
	let curtainRight: HTMLElement;
	let titleRef: HTMLElement;
	let subtitleRef: HTMLElement;
	let ctaRef: HTMLElement;
	let starsRef: HTMLElement;
	let nomineeRef: HTMLElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let W = (canvas.width = window.innerWidth);
		let H = (canvas.height = window.innerHeight);
		let raf: number;
		let time = 0;

		// Spotlight beams configuration
		type Beam = {
			angle: number; targetAngle: number; speed: number;
			x: number; width: number; alpha: number; hue: number;
		};

		const beams: Beam[] = [
			{ angle: -0.35, targetAngle: -0.2, speed: 0.004, x: W * 0.18, width: 0.12, alpha: 0.55, hue: 45 },
			{ angle: 0.1, targetAngle: 0.25, speed: 0.005, x: W * 0.82, width: 0.10, alpha: 0.5, hue: 50 },
			{ angle: -0.05, targetAngle: 0.0, speed: 0.003, x: W * 0.5, width: 0.08, alpha: 0.65, hue: 40 },
			{ angle: 0.3, targetAngle: 0.1, speed: 0.006, x: W * 0.32, width: 0.07, alpha: 0.4, hue: 55 },
			{ angle: -0.2, targetAngle: -0.35, speed: 0.0045, x: W * 0.68, width: 0.07, alpha: 0.4, hue: 42 },
		];

		// Dust motes floating in beams
		type Mote = { x: number; y: number; vx: number; vy: number; size: number; alpha: number };
		const motes: Mote[] = Array.from({ length: 180 }, () => ({
			x: Math.random() * W,
			y: Math.random() * H,
			vx: (Math.random() - 0.5) * 0.3,
			vy: -0.1 - Math.random() * 0.3,
			size: 0.5 + Math.random() * 1.5,
			alpha: 0.1 + Math.random() * 0.4,
		}));

		function drawBeam(b: Beam) {
			const bx = b.x;
			const spread = Math.tan(b.width) * H;
			const x1 = bx - spread / 2;
			const x2 = bx + spread / 2;

			// Oscillate angle
			b.angle += (b.targetAngle - b.angle) * 0.008;
			if (Math.abs(b.angle - b.targetAngle) < 0.005) {
				b.targetAngle = b.targetAngle + (Math.random() - 0.5) * 0.5;
				b.targetAngle = Math.max(-0.5, Math.min(0.5, b.targetAngle));
			}

			const pivotX = bx + Math.sin(b.angle) * H;
			const pivotX1 = pivotX - spread * 0.5;
			const pivotX2 = pivotX + spread * 0.5;

			const grad = ctx.createLinearGradient(bx, 0, bx, H);
			grad.addColorStop(0, `hsla(${b.hue},80%,85%,${b.alpha})`);
			grad.addColorStop(0.3, `hsla(${b.hue},70%,70%,${b.alpha * 0.5})`);
			grad.addColorStop(0.7, `hsla(${b.hue},60%,60%,${b.alpha * 0.2})`);
			grad.addColorStop(1, `hsla(${b.hue},60%,60%,0)`);

			ctx.save();
			ctx.beginPath();
			ctx.moveTo(bx, 0);
			ctx.lineTo(pivotX1, H);
			ctx.lineTo(pivotX2, H);
			ctx.closePath();
			ctx.fillStyle = grad;
			ctx.fill();
			ctx.restore();

			// Bright cone edge lines
			ctx.save();
			ctx.globalAlpha = b.alpha * 0.3;
			ctx.strokeStyle = `hsla(${b.hue},90%,95%,1)`;
			ctx.lineWidth = 0.5;
			ctx.beginPath();
			ctx.moveTo(bx, 0);
			ctx.lineTo(pivotX1, H);
			ctx.stroke();
			ctx.beginPath();
			ctx.moveTo(bx, 0);
			ctx.lineTo(pivotX2, H);
			ctx.stroke();
			ctx.restore();
		}

		function draw() {
			ctx.clearRect(0, 0, W, H);
			time += 0.016;

			// Stage ambient glow — floor
			const floorGrad = ctx.createLinearGradient(0, H * 0.7, 0, H);
			floorGrad.addColorStop(0, 'rgba(40,25,8,0.0)');
			floorGrad.addColorStop(1, 'rgba(20,12,4,0.6)');
			ctx.fillStyle = floorGrad;
			ctx.fillRect(0, H * 0.7, W, H * 0.3);

			// Stage floor reflection line
			ctx.save();
			ctx.globalAlpha = 0.25;
			const floorLine = ctx.createLinearGradient(0, 0, W, 0);
			floorLine.addColorStop(0, 'rgba(255,200,80,0)');
			floorLine.addColorStop(0.5, 'rgba(255,200,80,0.6)');
			floorLine.addColorStop(1, 'rgba(255,200,80,0)');
			ctx.strokeStyle = floorLine;
			ctx.lineWidth = 1;
			ctx.beginPath();
			ctx.moveTo(0, H * 0.78);
			ctx.lineTo(W, H * 0.78);
			ctx.stroke();
			ctx.restore();

			// Draw beams
			for (const b of beams) drawBeam(b);

			// Dust motes
			for (const m of motes) {
				m.x += m.vx;
				m.y += m.vy;
				if (m.y < -5) { m.y = H + 5; m.x = Math.random() * W; }
				if (m.x < 0 || m.x > W) m.x = Math.random() * W;
				ctx.save();
				ctx.globalAlpha = m.alpha * (0.5 + Math.sin(time * 2 + m.x) * 0.5);
				ctx.fillStyle = 'rgba(255, 220, 160, 1)';
				ctx.beginPath();
				ctx.arc(m.x, m.y, m.size, 0, Math.PI * 2);
				ctx.fill();
				ctx.restore();
			}

			// Central halo on stage
			const halo = ctx.createRadialGradient(W/2, H * 0.78, 0, W/2, H * 0.78, 280);
			halo.addColorStop(0, `rgba(255,210,100,${0.12 + Math.sin(time * 0.8) * 0.03})`);
			halo.addColorStop(0.4, `rgba(200,160,50,0.05)`);
			halo.addColorStop(1, 'rgba(0,0,0,0)');
			ctx.fillStyle = halo;
			ctx.fillRect(0, 0, W, H);

			raf = requestAnimationFrame(draw);
		}
		draw();

		// Curtain reveal
		gsap.set(curtainLeft, { x: '0%' });
		gsap.set(curtainRight, { x: '0%' });
		gsap.set([nomineeRef, titleRef, subtitleRef, ctaRef, starsRef], { autoAlpha: 0 });
		gsap.set(titleRef, { y: 60, scale: 0.9 });

		const tl = gsap.timeline({ delay: 0.3 });
		tl.to([curtainLeft, curtainRight], {
				x: (i) => (i === 0 ? '-100%' : '100%'),
				duration: 1.6,
				ease: 'expo.inOut',
			})
			.to(nomineeRef, { autoAlpha: 1, duration: 0.5, ease: 'power2.out' }, '-=0.5')
			.to(titleRef, { autoAlpha: 1, y: 0, scale: 1, duration: 1.2, ease: 'expo.out' }, '-=0.2')
			.to(starsRef, { autoAlpha: 1, duration: 0.8, ease: 'power2.out' }, '-=0.6')
			.to(subtitleRef, { autoAlpha: 1, duration: 0.7, ease: 'power2.out' }, '-=0.4')
			.to(ctaRef, { autoAlpha: 1, duration: 0.6, ease: 'power2.out' }, '-=0.4');

		// Applause sparkle stars
		const starEls = starsRef?.querySelectorAll<HTMLElement>('.h11__star');
		starEls?.forEach((s, i) => {
			gsap.to(s, {
				y: gsap.utils.random(-12, 12),
				x: gsap.utils.random(-6, 6),
				scale: gsap.utils.random(0.8, 1.3),
				duration: gsap.utils.random(1.5, 3),
				repeat: -1,
				yoyo: true,
				ease: 'sine.inOut',
				delay: i * 0.2,
			});
		});

		// Title oscillating glow
		gsap.to(titleRef, {
			filter: 'drop-shadow(0 0 60px rgba(255,200,80,0.7)) drop-shadow(0 0 120px rgba(255,160,30,0.3))',
			duration: 2.5,
			repeat: -1,
			yoyo: true,
			ease: 'sine.inOut',
			delay: 2,
		});

		const onResize = () => {
			W = canvas.width = window.innerWidth;
			H = canvas.height = window.innerHeight;
		};
		window.addEventListener('resize', onResize);

		return () => {
			cancelAnimationFrame(raf);
			window.removeEventListener('resize', onResize);
		};
	});
</script>

<section class="h11" bind:this={section}>
	<canvas bind:this={canvas}></canvas>

	<div class="h11__curtain h11__curtain-left" bind:this={curtainLeft}>
		<div class="h11__curtain-fold"></div>
		<div class="h11__curtain-fold"></div>
		<div class="h11__curtain-fold"></div>
	</div>
	<div class="h11__curtain h11__curtain-right" bind:this={curtainRight}>
		<div class="h11__curtain-fold"></div>
		<div class="h11__curtain-fold"></div>
		<div class="h11__curtain-fold"></div>
	</div>

	<div class="h11__content">
		<div class="h11__nominee" bind:this={nomineeRef}>
			<Trophy size={14} weight="fill" />
			<span>BEST DIRECTOR · ACADEMY AWARD WINNER</span>
			<Trophy size={14} weight="fill" />
		</div>

		<div class="h11__stars" bind:this={starsRef} aria-hidden="true">
			{#each { length: 7 } as _, i}
				<span class="h11__star" style="--i:{i}">
					<Star size={14 + i * 2} weight="fill" />
				</span>
			{/each}
		</div>

		<h1 class="h11__title" bind:this={titleRef}>
			<span class="h11__t1">AND THE</span>
			<span class="h11__t2">WINNER</span>
			<span class="h11__t3">IS...</span>
		</h1>

		<p class="h11__subtitle" bind:this={subtitleRef}>
			The most anticipated moment in cinematic history.
		</p>

		<div class="h11__cta" bind:this={ctaRef}>
			<button class="h11__btn">
				<span>Open the Envelope</span>
				<ArrowRight size={16} weight="bold" />
			</button>
		</div>
	</div>

	<div class="h11__vignette"></div>
	<div class="h11__floor-reflection"></div>
</section>

<style>
	.h11 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #0c0803;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	canvas {
		position: absolute;
		inset: 0;
		width: 100%; height: 100%;
		z-index: 2;
		pointer-events: none;
	}

	.h11__curtain {
		position: absolute;
		top: 0; bottom: 0;
		width: 52%;
		z-index: 20;
		overflow: hidden;
		display: flex;
	}

	.h11__curtain-left {
		left: 0;
		background: linear-gradient(90deg, #1a0a04 0%, #3d1408 40%, #6b2010 70%, #8b2a14 100%);
		box-shadow: inset -20px 0 60px rgba(0,0,0,0.6);
	}

	.h11__curtain-right {
		right: 0;
		background: linear-gradient(270deg, #1a0a04 0%, #3d1408 40%, #6b2010 70%, #8b2a14 100%);
		box-shadow: inset 20px 0 60px rgba(0,0,0,0.6);
	}

	.h11__curtain-fold {
		flex: 1;
		background: linear-gradient(90deg, rgba(0,0,0,0.3) 0%, rgba(0,0,0,0) 50%, rgba(0,0,0,0.2) 100%);
	}

	.h11__content {
		position: relative;
		z-index: 10;
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1.5rem;
	}

	.h11__nominee {
		display: inline-flex;
		align-items: center;
		gap: 0.6rem;
		font-family: var(--font-mono);
		font-size: 0.65rem;
		letter-spacing: 0.2em;
		text-transform: uppercase;
		color: rgba(255, 200, 80, 0.75);
		border-top: 1px solid rgba(255, 190, 60, 0.25);
		border-bottom: 1px solid rgba(255, 190, 60, 0.25);
		padding: 0.45rem 1rem;
	}

	.h11__stars {
		display: flex;
		align-items: center;
		gap: 0.4rem;
		color: rgba(255, 200, 80, 0.65);
	}

	.h11__star {
		display: inline-flex;
		color: rgba(255, calc(180 + var(--i) * 5), calc(40 + var(--i) * 4), calc(0.4 + var(--i) * 0.08));
		filter: drop-shadow(0 0 6px rgba(255, 200, 60, 0.4));
	}

	.h11__title {
		display: flex;
		flex-direction: column;
		align-items: center;
		line-height: 0.88;
		gap: 0.1em;
	}

	.h11__t1 {
		font-family: var(--font-cinematic);
		font-size: clamp(2rem, 4.5vw, 5rem);
		letter-spacing: 0.35em;
		color: rgba(255, 200, 100, 0.6);
		text-shadow: 0 0 30px rgba(255, 180, 50, 0.3);
	}

	.h11__t2 {
		font-family: var(--font-cinematic);
		font-size: clamp(6.5rem, 16vw, 18rem);
		letter-spacing: 0.06em;
		background: linear-gradient(160deg, #fff8e0 0%, #ffd060 30%, #ff9c20 60%, #e05010 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
		line-height: 0.82;
	}

	.h11__t3 {
		font-family: var(--font-cinematic);
		font-size: clamp(2.5rem, 5.5vw, 6.5rem);
		letter-spacing: 0.2em;
		color: transparent;
		-webkit-text-stroke: 1.5px rgba(255, 200, 80, 0.55);
		text-shadow: none;
	}

	.h11__subtitle {
		font-size: 0.9rem;
		color: rgba(220, 190, 130, 0.55);
		font-weight: 300;
		letter-spacing: 0.08em;
	}

	.h11__cta {
		margin-top: 0.5rem;
	}

	.h11__btn {
		display: inline-flex;
		align-items: center;
		gap: 0.6rem;
		padding: 0.9rem 2.4rem;
		background: linear-gradient(135deg, rgba(200, 140, 30, 0.18), rgba(150, 80, 10, 0.18));
		border: 1px solid rgba(255, 190, 60, 0.45);
		color: rgba(255, 220, 140, 0.95);
		font-size: 0.88rem;
		font-weight: 600;
		letter-spacing: 0.08em;
		cursor: pointer;
		backdrop-filter: blur(10px);
		transition: all 0.3s;
		box-shadow: 0 0 30px rgba(200, 150, 30, 0.2);
	}

	.h11__btn:hover {
		background: linear-gradient(135deg, rgba(220, 160, 40, 0.35), rgba(170, 90, 15, 0.35));
		box-shadow: 0 0 60px rgba(220, 160, 40, 0.45);
		transform: translateY(-2px) scale(1.02);
	}

	.h11__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at center 60%, transparent 30%, rgba(12,8,3,0.9) 100%);
		pointer-events: none;
		z-index: 5;
	}

	.h11__floor-reflection {
		position: absolute;
		bottom: 0; left: 0; right: 0;
		height: 22%;
		background: linear-gradient(to top, rgba(12,8,3,0.95) 0%, transparent 100%);
		pointer-events: none;
		z-index: 6;
	}
</style>
