<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { Lightning, ArrowRight } from 'phosphor-svelte';

	let section: HTMLElement;
	let canvas: HTMLCanvasElement;
	let titleRef: HTMLElement;
	let subtitleRef: HTMLElement;
	let ctaRef: HTMLElement;
	let eyebrowRef: HTMLElement;
	let speedoRef: HTMLElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let W = (canvas.width = window.innerWidth);
		let H = (canvas.height = window.innerHeight);
		let raf: number;
		let time = 0;
		let speedVal = 0;
		let targetSpeed = 0.85;

		// Motion streak lines
		type Streak = {
			x: number; y: number; len: number;
			speed: number; alpha: number; width: number;
			hue: number; cyOff: number;
		};

		const STREAK_COUNT = 220;
		const streaks: Streak[] = Array.from({ length: STREAK_COUNT }, (_, i) => ({
			x: Math.random() * W,
			y: Math.random() * H,
			len: 40 + Math.random() * 260,
			speed: 18 + Math.random() * 55,
			alpha: 0.08 + Math.random() * 0.55,
			width: 0.5 + Math.random() * 2,
			hue: Math.random() > 0.6 ? 200 : Math.random() > 0.4 ? 180 : 220,
			cyOff: (Math.random() - 0.5) * H,
		}));

		// Soundwave bars
		const BAR_COUNT = 80;
		type Bar = { phase: number; speed: number; baseH: number };
		const bars: Bar[] = Array.from({ length: BAR_COUNT }, (_, i) => ({
			phase: (i / BAR_COUNT) * Math.PI * 2,
			speed: 2 + Math.random() * 3,
			baseH: 10 + Math.random() * 30,
		}));

		function draw() {
			ctx.fillStyle = 'rgba(2,4,12,0.22)';
			ctx.fillRect(0, 0, W, H);
			time += 0.016;
			speedVal += (targetSpeed - speedVal) * 0.03;

			const cx = W / 2;
			const cy = H / 2;

			// Speed tunnel vanishing point
			const tunnelLayers = 12;
			for (let i = tunnelLayers; i > 0; i--) {
				const t = i / tunnelLayers;
				const size = t * Math.min(W, H) * 0.7;
				const alpha = (1 - t) * speedVal * 0.12;
				ctx.strokeStyle = `rgba(60,120,255,${alpha})`;
				ctx.lineWidth = 1;
				ctx.strokeRect(cx - size / 2, cy - size / 2, size, size * 0.55);
			}

			// Radial speed lines from center
			const lineCount = 60;
			for (let i = 0; i < lineCount; i++) {
				const angle = (i / lineCount) * Math.PI * 2 + time * 0.5;
				const innerR = 80 + Math.sin(time * 2 + i) * 10;
				const outerR = innerR + (120 + Math.random() * 180) * speedVal;
				const x1 = cx + Math.cos(angle) * innerR;
				const y1 = cy + Math.sin(angle) * innerR * 0.45;
				const x2 = cx + Math.cos(angle) * outerR;
				const y2 = cy + Math.sin(angle) * outerR * 0.45;
				const alpha = (0.04 + Math.random() * 0.06) * speedVal;
				ctx.beginPath();
				ctx.moveTo(x1, y1);
				ctx.lineTo(x2, y2);
				ctx.strokeStyle = `rgba(80,150,255,${alpha})`;
				ctx.lineWidth = 0.8;
				ctx.stroke();
			}

			// Horizontal motion streaks
			for (const s of streaks) {
				s.x -= s.speed * speedVal * 1.8;
				if (s.x + s.len < 0) {
					s.x = W + s.len;
					s.y = Math.random() * H;
				}
				const grad = ctx.createLinearGradient(s.x - s.len, s.y, s.x, s.y);
				grad.addColorStop(0, `hsla(${s.hue},80%,70%,0)`);
				grad.addColorStop(0.6, `hsla(${s.hue},80%,70%,${s.alpha * speedVal})`);
				grad.addColorStop(1, `hsla(${s.hue},90%,85%,${s.alpha * speedVal * 0.6})`);
				ctx.beginPath();
				ctx.moveTo(s.x - s.len, s.y);
				ctx.lineTo(s.x, s.y);
				ctx.strokeStyle = grad;
				ctx.lineWidth = s.width;
				ctx.stroke();
			}

			// Sound wave bars at bottom
			const barW = W / BAR_COUNT;
			const waveY = H * 0.82;
			for (let i = 0; i < BAR_COUNT; i++) {
				const b = bars[i];
				const barH = (b.baseH + Math.sin(time * b.speed + b.phase) * b.baseH * 1.4) * (0.4 + speedVal * 0.8);
				const bx = i * barW;
				const alpha = 0.3 + Math.sin(time * b.speed + b.phase) * 0.2;
				const hue = 200 + (i / BAR_COUNT) * 40;
				const grad = ctx.createLinearGradient(bx, waveY, bx, waveY - barH);
				grad.addColorStop(0, `hsla(${hue},80%,60%,0.05)`);
				grad.addColorStop(1, `hsla(${hue},90%,70%,${alpha})`);
				ctx.fillStyle = grad;
				ctx.fillRect(bx + 1, waveY - barH, barW - 2, barH);
			}

			// Central lens flare
			const lensAlpha = 0.06 + Math.sin(time * 3) * 0.02;
			const lens = ctx.createRadialGradient(cx, cy, 0, cx, cy, 180);
			lens.addColorStop(0, `rgba(140,180,255,${lensAlpha * 3})`);
			lens.addColorStop(0.3, `rgba(80,120,220,${lensAlpha})`);
			lens.addColorStop(1, 'rgba(0,0,0,0)');
			ctx.fillStyle = lens;
			ctx.beginPath();
			ctx.arc(cx, cy, 180, 0, Math.PI * 2);
			ctx.fill();

			// Speed number overlay
			if (speedoRef) {
				const displaySpeed = Math.round(speedVal * 1099 + 1);
				speedoRef.textContent = displaySpeed.toLocaleString() + ' mph';
			}

			raf = requestAnimationFrame(draw);
		}
		draw();

		// Entrance
		gsap.set([eyebrowRef, titleRef, subtitleRef, ctaRef], { autoAlpha: 0 });
		gsap.set(titleRef, { x: -80, skewX: -8 });
		gsap.set(subtitleRef, { x: -40 });
		gsap.set(ctaRef, { x: -30 });

		const tl = gsap.timeline({ delay: 0.2 });
		tl.to(eyebrowRef, { autoAlpha: 1, duration: 0.5, ease: 'power2.out' })
			.to(titleRef, { autoAlpha: 1, x: 0, skewX: 0, duration: 0.9, ease: 'expo.out' }, '-=0.2')
			.to(subtitleRef, { autoAlpha: 1, x: 0, duration: 0.8, ease: 'expo.out' }, '-=0.5')
			.to(ctaRef, { autoAlpha: 1, x: 0, duration: 0.7, ease: 'expo.out' }, '-=0.45');

		// Speed ramp up
		gsap.fromTo({ v: 0 }, { v: 0 }, {
			v: 1,
			duration: 3,
			ease: 'power2.in',
			delay: 0.5,
			onUpdate: function () {
				targetSpeed = this.targets()[0].v;
			},
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

<section class="h10" bind:this={section}>
	<canvas bind:this={canvas}></canvas>
	<div class="h10__bg"></div>

	<div class="h10__content">
		<div class="h10__eyebrow" bind:this={eyebrowRef}>
			<Lightning size={13} weight="fill" />
			<span>VELOCITY · UNRESTRICTED</span>
		</div>

		<h1 class="h10__title" bind:this={titleRef}>
			<span class="h10__t1">BREAK</span>
			<span class="h10__t2">EVERY</span>
			<span class="h10__t3">LIMIT</span>
		</h1>

		<div class="h10__speedo" bind:this={speedoRef}>0 mph</div>

		<p class="h10__subtitle" bind:this={subtitleRef}>
			Zero to impossible in 1.8 seconds.
		</p>

		<div class="h10__cta" bind:this={ctaRef}>
			<button class="h10__btn">
				<Lightning size={16} weight="fill" />
				<span>Full Throttle</span>
				<ArrowRight size={16} weight="bold" />
			</button>
		</div>
	</div>

	<div class="h10__vignette"></div>
	<div class="h10__scanlines"></div>
</section>

<style>
	.h10 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #02040c;
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

	.h10__bg {
		position: absolute;
		inset: 0;
		background:
			radial-gradient(ellipse at 50% 50%, rgba(30, 60, 160, 0.14) 0%, transparent 55%);
		z-index: 1;
	}

	.h10__content {
		position: relative;
		z-index: 10;
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1.2rem;
	}

	.h10__eyebrow {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		font-family: var(--font-mono);
		font-size: 0.68rem;
		letter-spacing: 0.22em;
		text-transform: uppercase;
		color: rgba(100, 160, 255, 0.75);
		border-bottom: 1px solid rgba(80, 140, 255, 0.25);
		padding-bottom: 0.4rem;
	}

	.h10__title {
		display: flex;
		flex-direction: column;
		align-items: flex-start;
		line-height: 0.86;
	}

	.h10__t1 {
		font-family: var(--font-cinematic);
		font-size: clamp(3rem, 7vw, 8rem);
		letter-spacing: 0.12em;
		color: rgba(160, 190, 255, 0.7);
		text-shadow: 0 0 40px rgba(80, 140, 255, 0.4);
		align-self: flex-start;
	}

	.h10__t2 {
		font-family: var(--font-cinematic);
		font-size: clamp(6rem, 16vw, 18rem);
		letter-spacing: 0.02em;
		color: #fff;
		text-shadow:
			0 0 20px rgba(100, 160, 255, 0.8),
			0 0 60px rgba(60, 120, 220, 0.5),
			-8px 0 30px rgba(255, 80, 80, 0.15),
			8px 0 30px rgba(80, 200, 255, 0.2);
		line-height: 0.85;
	}

	.h10__t3 {
		font-family: var(--font-cinematic);
		font-size: clamp(3rem, 7vw, 8rem);
		letter-spacing: 0.28em;
		color: transparent;
		-webkit-text-stroke: 1px rgba(100, 160, 255, 0.45);
		align-self: flex-end;
	}

	.h10__speedo {
		font-family: var(--font-mono);
		font-size: clamp(1.1rem, 2.5vw, 2rem);
		font-weight: 700;
		color: rgba(100, 180, 255, 0.9);
		letter-spacing: 0.1em;
		text-shadow: 0 0 20px rgba(80, 160, 255, 0.6);
	}

	.h10__subtitle {
		font-size: 0.9rem;
		color: rgba(140, 170, 230, 0.55);
		font-weight: 300;
		letter-spacing: 0.1em;
		text-transform: uppercase;
	}

	.h10__cta {
		margin-top: 0.5rem;
	}

	.h10__btn {
		display: inline-flex;
		align-items: center;
		gap: 0.55rem;
		padding: 0.9rem 2.4rem;
		background: linear-gradient(135deg, rgba(60, 120, 255, 0.2), rgba(30, 80, 200, 0.2));
		border: 1px solid rgba(80, 150, 255, 0.5);
		color: rgba(180, 210, 255, 0.95);
		font-size: 0.9rem;
		font-weight: 700;
		letter-spacing: 0.12em;
		text-transform: uppercase;
		cursor: pointer;
		backdrop-filter: blur(10px);
		transition: all 0.25s;
		box-shadow: 0 0 30px rgba(60, 120, 255, 0.2), -4px 0 20px rgba(255, 80, 80, 0.05);
	}

	.h10__btn:hover {
		background: linear-gradient(135deg, rgba(80, 140, 255, 0.38), rgba(40, 100, 220, 0.38));
		box-shadow: 0 0 60px rgba(80, 150, 255, 0.5);
		transform: translateX(3px);
	}

	.h10__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at center, transparent 28%, rgba(2,4,12,0.9) 100%);
		pointer-events: none;
		z-index: 5;
	}

	.h10__scanlines {
		position: absolute;
		inset: 0;
		background: repeating-linear-gradient(
			0deg,
			transparent, transparent 3px,
			rgba(0,0,0,0.05) 3px, rgba(0,0,0,0.05) 4px
		);
		pointer-events: none;
		z-index: 6;
	}
</style>
