<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { CloudLightning, ArrowRight } from 'phosphor-svelte';

	let section: HTMLElement;
	let canvas: HTMLCanvasElement;
	let titleRef: HTMLElement;
	let subtitleRef: HTMLElement;
	let ctaRef: HTMLElement;
	let eyebrowRef: HTMLElement;

	type LightningBolt = {
		points: [number, number][];
		alpha: number;
		width: number;
		color: string;
		children: LightningBolt[];
	};

	function buildLightning(
		x1: number, y1: number, x2: number, y2: number,
		roughness: number, depth: number,
		color = '#ffffff', width = 2.5
	): LightningBolt {
		const points: [number, number][] = [[x1, y1]];
		const segments = 12;
		for (let i = 1; i < segments; i++) {
			const t = i / segments;
			const mx = x1 + (x2 - x1) * t + (Math.random() - 0.5) * roughness;
			const my = y1 + (y2 - y1) * t + (Math.random() - 0.5) * roughness * 0.5;
			points.push([mx, my]);
		}
		points.push([x2, y2]);

		const children: LightningBolt[] = [];
		if (depth > 0) {
			const branchCount = 1 + Math.floor(Math.random() * 2);
			for (let b = 0; b < branchCount; b++) {
				const branchIdx = 3 + Math.floor(Math.random() * (points.length - 4));
				const [bx, by] = points[branchIdx];
				const angle = Math.atan2(y2 - y1, x2 - x1) + (Math.random() - 0.5) * 1.4;
				const len = 80 + Math.random() * 140;
				children.push(buildLightning(
					bx, by,
					bx + Math.cos(angle) * len,
					by + Math.sin(angle) * len,
					roughness * 0.55, depth - 1,
					color, width * 0.45
				));
			}
		}
		return { points, alpha: 1, width, color, children };
	}

	function drawBolt(ctx: CanvasRenderingContext2D, bolt: LightningBolt) {
		if (bolt.alpha <= 0) return;
		ctx.save();
		ctx.globalAlpha = bolt.alpha;
		ctx.lineWidth = bolt.width;
		ctx.strokeStyle = bolt.color;
		ctx.shadowColor = bolt.color;
		ctx.shadowBlur = 20;
		ctx.beginPath();
		ctx.moveTo(bolt.points[0][0], bolt.points[0][1]);
		for (let i = 1; i < bolt.points.length; i++) {
			ctx.lineTo(bolt.points[i][0], bolt.points[i][1]);
		}
		ctx.stroke();

		// bright core
		ctx.lineWidth = bolt.width * 0.35;
		ctx.strokeStyle = '#ffffff';
		ctx.shadowBlur = 6;
		ctx.stroke();
		ctx.restore();

		for (const child of bolt.children) {
			child.alpha = bolt.alpha;
			drawBolt(ctx, child);
		}
	}

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let W = (canvas.width = window.innerWidth);
		let H = (canvas.height = window.innerHeight);
		let raf: number;

		type ActiveBolt = { bolt: LightningBolt; life: number; maxLife: number };
		const activeBolts: ActiveBolt[] = [];
		let cloudFlash = 0;
		let time = 0;

		function spawnBolt() {
			const x = W * 0.1 + Math.random() * W * 0.8;
			const y = 0;
			const tx = x + (Math.random() - 0.5) * 300;
			const ty = H * 0.3 + Math.random() * H * 0.55;

			const hue = Math.random() > 0.7 ? '200,220,255' : Math.random() > 0.5 ? '180,140,255' : '255,255,255';
			const bolt = buildLightning(x, y, tx, ty, 120, 3, `rgb(${hue})`, 2.5 + Math.random() * 2);
			activeBolts.push({ bolt, life: 1, maxLife: 1 });
			cloudFlash = 0.6 + Math.random() * 0.4;
		}

		function draw() {
			ctx.clearRect(0, 0, W, H);
			time += 0.016;

			// Storm cloud atmosphere
			const cloudGrad = ctx.createLinearGradient(0, 0, 0, H * 0.45);
			cloudGrad.addColorStop(0, `rgba(8,6,22,${0.96 + Math.sin(time * 0.4) * 0.02})`);
			cloudGrad.addColorStop(0.6, 'rgba(14,10,30,0.88)');
			cloudGrad.addColorStop(1, 'rgba(0,0,0,0)');
			ctx.fillStyle = cloudGrad;
			ctx.fillRect(0, 0, W, H * 0.5);

			// Cloud flash
			if (cloudFlash > 0) {
				const flashGrad = ctx.createRadialGradient(W/2, 0, 0, W/2, 0, W * 0.8);
				flashGrad.addColorStop(0, `rgba(140,120,255,${cloudFlash * 0.35})`);
				flashGrad.addColorStop(0.4, `rgba(60,40,120,${cloudFlash * 0.18})`);
				flashGrad.addColorStop(1, 'rgba(0,0,0,0)');
				ctx.fillStyle = flashGrad;
				ctx.fillRect(0, 0, W, H);
				cloudFlash -= 0.045;
				if (cloudFlash < 0) cloudFlash = 0;
			}

			// Rain streaks
			ctx.save();
			ctx.strokeStyle = 'rgba(140,160,220,0.12)';
			ctx.lineWidth = 0.8;
			const rainCount = 120;
			for (let i = 0; i < rainCount; i++) {
				const rx = ((i * 137.5 + time * 280) % W);
				const ry = ((i * 73 + time * 420) % H);
				ctx.beginPath();
				ctx.moveTo(rx, ry);
				ctx.lineTo(rx - 4, ry + 18);
				ctx.stroke();
			}
			ctx.restore();

			// Draw and decay bolts
			for (let i = activeBolts.length - 1; i >= 0; i--) {
				const ab = activeBolts[i];
				ab.bolt.alpha = ab.life;
				drawBolt(ctx, ab.bolt);
				ab.life -= 0.055;
				if (ab.life <= 0) activeBolts.splice(i, 1);
			}

			// Ground glow on strike
			if (activeBolts.length > 0) {
				const last = activeBolts[activeBolts.length - 1];
				const gp = last.bolt.points[last.bolt.points.length - 1];
				const gg = ctx.createRadialGradient(gp[0], gp[1], 0, gp[0], gp[1], 120);
				gg.addColorStop(0, `rgba(180,160,255,${last.life * 0.5})`);
				gg.addColorStop(1, 'rgba(0,0,0,0)');
				ctx.fillStyle = gg;
				ctx.fillRect(0, 0, W, H);
			}

			raf = requestAnimationFrame(draw);
		}
		draw();

		// Spawn bolts on irregular intervals
		let nextBolt = 0.4;
		const boltTimer = setInterval(() => {
			spawnBolt();
			if (Math.random() > 0.6) setTimeout(spawnBolt, 80);
			if (Math.random() > 0.8) setTimeout(spawnBolt, 160);
		}, 600 + Math.random() * 1400);

		// Entrance animations
		gsap.set([eyebrowRef, titleRef, subtitleRef, ctaRef], { autoAlpha: 0, y: 40 });
		const tl = gsap.timeline({ delay: 0.3 });
		tl.to(eyebrowRef, { autoAlpha: 1, y: 0, duration: 0.6, ease: 'power3.out' })
			.to(titleRef, { autoAlpha: 1, y: 0, duration: 1.0, ease: 'power3.out' }, '-=0.2')
			.to(subtitleRef, { autoAlpha: 1, y: 0, duration: 0.8, ease: 'power3.out' }, '-=0.5')
			.to(ctaRef, { autoAlpha: 1, y: 0, duration: 0.7, ease: 'power3.out' }, '-=0.4');

		// Title electric pulse sync with bolts
		const pulseInterval = setInterval(() => {
			gsap.fromTo(titleRef,
				{ textShadow: '0 0 80px rgba(140,120,255,0.9), 0 0 160px rgba(100,80,200,0.5)' },
				{ textShadow: '0 0 20px rgba(140,120,255,0.2)', duration: 0.5, ease: 'power2.out' }
			);
		}, 700 + Math.random() * 800);

		const onResize = () => {
			W = canvas.width = window.innerWidth;
			H = canvas.height = window.innerHeight;
		};
		window.addEventListener('resize', onResize);

		return () => {
			cancelAnimationFrame(raf);
			clearInterval(boltTimer);
			clearInterval(pulseInterval);
			window.removeEventListener('resize', onResize);
		};
	});
</script>

<section class="h8" bind:this={section}>
	<canvas bind:this={canvas}></canvas>

	<div class="h8__atmosphere"></div>

	<div class="h8__content">
		<div class="h8__eyebrow" bind:this={eyebrowRef}>
			<CloudLightning size={14} weight="fill" />
			<span>CATEGORY 5 · STORM SEASON</span>
		</div>

		<h1 class="h8__title" bind:this={titleRef}>
			<span class="h8__t1">THE</span>
			<span class="h8__t2">STORM</span>
			<span class="h8__t3">WITHIN</span>
		</h1>

		<p class="h8__subtitle" bind:this={subtitleRef}>
			10,000 volts of pure cinematic fury.
		</p>

		<div class="h8__cta" bind:this={ctaRef}>
			<button class="h8__btn">
				<span>Enter the Eye</span>
				<ArrowRight size={16} weight="bold" />
			</button>
			<div class="h8__weather-data">
				<span>⚡ 847 strikes/min</span>
				<span>·</span>
				<span>Wind 182mph</span>
			</div>
		</div>
	</div>

	<div class="h8__vignette"></div>
	<div class="h8__ground-fog"></div>
</section>

<style>
	.h8 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #04030e;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	canvas {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		z-index: 2;
		pointer-events: none;
	}

	.h8__atmosphere {
		position: absolute;
		inset: 0;
		background:
			radial-gradient(ellipse at 50% 0%, rgba(30,20,80,0.6) 0%, transparent 60%),
			radial-gradient(ellipse at 50% 100%, rgba(0,0,20,0.9) 0%, transparent 50%);
		z-index: 1;
	}

	.h8__content {
		position: relative;
		z-index: 10;
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1.6rem;
	}

	.h8__eyebrow {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		font-family: var(--font-mono);
		font-size: 0.68rem;
		letter-spacing: 0.2em;
		text-transform: uppercase;
		color: rgba(160, 140, 255, 0.8);
		border: 1px solid rgba(140, 120, 255, 0.25);
		padding: 0.3rem 0.9rem;
	}

	.h8__title {
		display: flex;
		flex-direction: column;
		align-items: center;
		line-height: 0.88;
	}

	.h8__t1 {
		font-family: var(--font-cinematic);
		font-size: clamp(2.5rem, 6vw, 6rem);
		letter-spacing: 0.5em;
		color: rgba(200, 190, 255, 0.7);
		text-shadow: 0 0 30px rgba(140, 120, 255, 0.4);
	}

	.h8__t2 {
		font-family: var(--font-cinematic);
		font-size: clamp(7rem, 18vw, 20rem);
		letter-spacing: 0.04em;
		color: #fff;
		text-shadow:
			0 0 30px rgba(140, 120, 255, 0.7),
			0 0 80px rgba(100, 80, 200, 0.4),
			0 0 160px rgba(60, 40, 140, 0.2);
		line-height: 0.85;
	}

	.h8__t3 {
		font-family: var(--font-cinematic);
		font-size: clamp(2rem, 4.5vw, 5rem);
		letter-spacing: 0.45em;
		color: transparent;
		-webkit-text-stroke: 1px rgba(160, 140, 255, 0.5);
	}

	.h8__subtitle {
		font-size: 0.95rem;
		color: rgba(180, 170, 230, 0.55);
		font-weight: 300;
		letter-spacing: 0.1em;
		text-transform: uppercase;
	}

	.h8__cta {
		display: flex;
		align-items: center;
		gap: 1.4rem;
	}

	.h8__btn {
		display: inline-flex;
		align-items: center;
		gap: 0.6rem;
		padding: 0.88rem 2.1rem;
		background: rgba(100, 80, 200, 0.18);
		border: 1px solid rgba(140, 120, 255, 0.45);
		color: rgba(210, 200, 255, 0.95);
		font-size: 0.88rem;
		font-weight: 600;
		letter-spacing: 0.06em;
		cursor: pointer;
		backdrop-filter: blur(10px);
		transition: all 0.25s;
		box-shadow: 0 0 30px rgba(100, 80, 200, 0.2);
	}

	.h8__btn:hover {
		background: rgba(120, 100, 220, 0.35);
		box-shadow: 0 0 60px rgba(140, 120, 255, 0.45);
		transform: translateY(-2px);
	}

	.h8__weather-data {
		display: flex;
		gap: 0.5rem;
		font-family: var(--font-mono);
		font-size: 0.7rem;
		color: rgba(140, 130, 200, 0.45);
		letter-spacing: 0.06em;
	}

	.h8__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at center, transparent 25%, rgba(4,3,14,0.85) 100%);
		pointer-events: none;
		z-index: 5;
	}

	.h8__ground-fog {
		position: absolute;
		bottom: 0;
		left: 0; right: 0;
		height: 30%;
		background: linear-gradient(to top, rgba(4,3,14,0.95) 0%, rgba(10,8,30,0.4) 50%, transparent 100%);
		pointer-events: none;
		z-index: 6;
	}
</style>
