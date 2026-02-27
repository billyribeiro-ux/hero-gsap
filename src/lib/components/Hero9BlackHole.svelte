<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { Planet, ArrowRight } from 'phosphor-svelte';

	let section: HTMLElement;
	let canvas: HTMLCanvasElement;
	let titleRef: HTMLElement;
	let subtitleRef: HTMLElement;
	let ctaRef: HTMLElement;
	let eyebrowRef: HTMLElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let W = (canvas.width = window.innerWidth);
		let H = (canvas.height = window.innerHeight);
		let raf: number;
		let time = 0;

		// Accretion disk particles
		type DiskParticle = {
			angle: number; radius: number; speed: number;
			size: number; brightness: number; hue: number; z: number;
		};

		const DISK_COUNT = 900;
		const disk: DiskParticle[] = Array.from({ length: DISK_COUNT }, () => {
			const r = 100 + Math.random() * 340;
			return {
				angle: Math.random() * Math.PI * 2,
				radius: r,
				speed: (0.008 + Math.random() * 0.018) * (200 / r),
				size: 0.5 + Math.random() * 2.2,
				brightness: 0.3 + Math.random() * 0.7,
				hue: Math.random() > 0.6 ? 30 + Math.random() * 30 : Math.random() > 0.5 ? 190 + Math.random() * 40 : 0,
				z: (Math.random() - 0.5) * 0.35,
			};
		});

		// Debris/stars sucked in
		type Debris = { x: number; y: number; angle: number; dist: number; speed: number; alpha: number; size: number };
		const debris: Debris[] = Array.from({ length: 200 }, () => ({
			x: 0, y: 0,
			angle: Math.random() * Math.PI * 2,
			dist: 250 + Math.random() * 500,
			speed: 0.003 + Math.random() * 0.006,
			alpha: Math.random(),
			size: Math.random() * 1.5,
		}));

		function draw() {
			ctx.fillStyle = 'rgba(0,0,4,0.18)';
			ctx.fillRect(0, 0, W, H);
			time += 0.016;

			const cx = W / 2, cy = H / 2;
			const BH_R = 72; // event horizon radius

			// Gravitational lensing glow rings
			for (let ring = 6; ring > 0; ring--) {
				const rr = BH_R + ring * 22;
				const alpha = (0.04 - ring * 0.004) * (1 + Math.sin(time * 0.5) * 0.2);
				const grad = ctx.createRadialGradient(cx, cy, rr * 0.7, cx, cy, rr + 18);
				grad.addColorStop(0, `rgba(255,140,20,${alpha * 3})`);
				grad.addColorStop(0.5, `rgba(255,80,0,${alpha * 1.5})`);
				grad.addColorStop(1, 'rgba(0,0,0,0)');
				ctx.beginPath();
				ctx.arc(cx, cy, rr, 0, Math.PI * 2);
				ctx.strokeStyle = `rgba(255,${100 + ring * 8},0,${alpha * 2})`;
				ctx.lineWidth = 1.5;
				ctx.stroke();
			}

			// Accretion disk — drawn as ellipse perspective
			ctx.save();
			ctx.translate(cx, cy);
			ctx.scale(1, 0.28); // perspective flatten

			for (const p of disk) {
				p.angle += p.speed;
				const px = Math.cos(p.angle) * p.radius;
				const py = Math.sin(p.angle) * p.radius;

				// Hide behind black hole
				const behind = Math.sin(p.angle) < 0;
				const distFactor = Math.max(0, 1 - BH_R / p.radius);
				const alphaMul = behind ? distFactor * 0.45 : distFactor;

				const sat = p.hue === 0 ? '0%' : '90%';
				ctx.globalAlpha = p.brightness * alphaMul;
				ctx.fillStyle = `hsl(${p.hue},${sat},${60 + p.brightness * 35}%)`;
				ctx.shadowColor = `hsl(${p.hue},90%,70%)`;
				ctx.shadowBlur = 6;
				ctx.beginPath();
				ctx.arc(px, py, p.size, 0, Math.PI * 2);
				ctx.fill();
			}
			ctx.restore();

			// Debris spiral inward
			for (const d of debris) {
				d.angle += d.speed;
				d.dist -= 0.08 + (500 / Math.max(50, d.dist)) * 0.12;
				if (d.dist < BH_R * 0.8) {
					d.dist = 320 + Math.random() * 320;
					d.angle = Math.random() * Math.PI * 2;
					d.alpha = Math.random() * 0.5;
				}
				const dx = cx + Math.cos(d.angle) * d.dist;
				const dy = cy + Math.sin(d.angle) * d.dist * 0.55;
				const fadeFactor = Math.min(1, (d.dist - BH_R) / 80);
				ctx.globalAlpha = d.alpha * fadeFactor * 0.7;
				ctx.fillStyle = 'rgba(200,210,255,1)';
				ctx.shadowColor = 'rgba(180,200,255,0.8)';
				ctx.shadowBlur = 4;
				ctx.beginPath();
				ctx.arc(dx, dy, d.size, 0, Math.PI * 2);
				ctx.fill();
			}

			ctx.globalAlpha = 1;

			// Photon sphere ring
			const photonGrad = ctx.createRadialGradient(cx, cy, BH_R * 1.35, cx, cy, BH_R * 1.65);
			photonGrad.addColorStop(0, `rgba(255,200,80,${0.15 + Math.sin(time * 1.2) * 0.05})`);
			photonGrad.addColorStop(1, 'rgba(0,0,0,0)');
			ctx.fillStyle = photonGrad;
			ctx.beginPath();
			ctx.arc(cx, cy, BH_R * 1.6, 0, Math.PI * 2);
			ctx.fill();

			// Event horizon — absolute black
			const bhGrad = ctx.createRadialGradient(cx, cy, 0, cx, cy, BH_R * 1.1);
			bhGrad.addColorStop(0, 'rgba(0,0,0,1)');
			bhGrad.addColorStop(0.8, 'rgba(0,0,0,1)');
			bhGrad.addColorStop(1, 'rgba(0,0,0,0)');
			ctx.fillStyle = bhGrad;
			ctx.beginPath();
			ctx.arc(cx, cy, BH_R * 1.1, 0, Math.PI * 2);
			ctx.fill();

			// Jet streams (relativistic jets)
			for (const dir of [-1, 1]) {
				const jetGrad = ctx.createLinearGradient(cx, cy, cx, cy + dir * H * 0.55);
				jetGrad.addColorStop(0, `rgba(180,160,255,${0.35 + Math.sin(time * 2) * 0.08})`);
				jetGrad.addColorStop(0.3, `rgba(100,80,200,0.15)`);
				jetGrad.addColorStop(1, 'rgba(0,0,0,0)');
				ctx.save();
				ctx.translate(cx, cy);
				ctx.beginPath();
				ctx.moveTo(-8, 0);
				ctx.lineTo(8, 0);
				ctx.lineTo(2 + Math.sin(time) * 3, dir * H * 0.55);
				ctx.lineTo(-2 - Math.sin(time + 1) * 3, dir * H * 0.55);
				ctx.closePath();
				ctx.fillStyle = jetGrad;
				ctx.fill();
				ctx.restore();
			}

			raf = requestAnimationFrame(draw);
		}
		draw();

		// Entrance
		gsap.set([eyebrowRef, titleRef, subtitleRef, ctaRef], { autoAlpha: 0, y: 35 });
		const tl = gsap.timeline({ delay: 0.4 });
		tl.to(eyebrowRef, { autoAlpha: 1, y: 0, duration: 0.7, ease: 'power3.out' })
			.to(titleRef, { autoAlpha: 1, y: 0, duration: 1.1, ease: 'power3.out' }, '-=0.3')
			.to(subtitleRef, { autoAlpha: 1, y: 0, duration: 0.9, ease: 'power3.out' }, '-=0.55')
			.to(ctaRef, { autoAlpha: 1, y: 0, duration: 0.8, ease: 'power3.out' }, '-=0.45');

		// Slow title suck effect
		gsap.to(titleRef, {
			scale: 0.97,
			duration: 8,
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

<section class="h9" bind:this={section}>
	<canvas bind:this={canvas}></canvas>

	<div class="h9__content">
		<div class="h9__eyebrow" bind:this={eyebrowRef}>
			<Planet size={13} weight="fill" />
			<span>INTERSTELLAR CLASSIFIED · SECTOR OMEGA</span>
		</div>

		<h1 class="h9__title" bind:this={titleRef}>
			<span class="h9__t1">EVENT</span>
			<span class="h9__t2">HORIZON</span>
		</h1>

		<p class="h9__subtitle" bind:this={subtitleRef}>
			Beyond this point, not even light escapes.<br />
			Neither will you.
		</p>

		<div class="h9__cta" bind:this={ctaRef}>
			<button class="h9__btn">
				Cross the Threshold
				<ArrowRight size={16} weight="bold" />
			</button>
			<div class="h9__data">
				<span>Mass: 4.3M☉</span>
				<span>·</span>
				<span>Spin: 0.998a</span>
			</div>
		</div>
	</div>

	<div class="h9__vignette"></div>
</section>

<style>
	.h9 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #000004;
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

	.h9__content {
		position: relative;
		z-index: 10;
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1.8rem;
		margin-top: 30vh;
	}

	.h9__eyebrow {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		font-family: var(--font-mono);
		font-size: 0.66rem;
		letter-spacing: 0.2em;
		text-transform: uppercase;
		color: rgba(255, 180, 80, 0.7);
		border: 1px solid rgba(255, 160, 40, 0.2);
		padding: 0.3rem 0.9rem;
	}

	.h9__title {
		display: flex;
		flex-direction: column;
		align-items: center;
		line-height: 0.9;
	}

	.h9__t1 {
		font-family: var(--font-cinematic);
		font-size: clamp(2rem, 5vw, 5.5rem);
		letter-spacing: 0.55em;
		color: rgba(255, 200, 120, 0.65);
		text-shadow: 0 0 40px rgba(255, 150, 50, 0.4);
	}

	.h9__t2 {
		font-family: var(--font-cinematic);
		font-size: clamp(5rem, 13vw, 14rem);
		letter-spacing: 0.04em;
		background: linear-gradient(180deg, #fff 0%, #ffd080 40%, #ff8020 70%, rgba(180,60,0,0.6) 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
		filter: drop-shadow(0 0 40px rgba(255, 140, 30, 0.6));
	}

	.h9__subtitle {
		font-size: 0.93rem;
		color: rgba(220, 190, 140, 0.55);
		font-weight: 300;
		line-height: 1.8;
		letter-spacing: 0.04em;
	}

	.h9__cta {
		display: flex;
		align-items: center;
		gap: 1.4rem;
	}

	.h9__btn {
		display: inline-flex;
		align-items: center;
		gap: 0.6rem;
		padding: 0.88rem 2.1rem;
		background: rgba(255, 140, 30, 0.12);
		border: 1px solid rgba(255, 150, 40, 0.4);
		color: rgba(255, 210, 140, 0.95);
		font-size: 0.88rem;
		font-weight: 600;
		letter-spacing: 0.05em;
		cursor: pointer;
		backdrop-filter: blur(10px);
		transition: all 0.25s;
		box-shadow: 0 0 30px rgba(255, 140, 30, 0.15);
	}

	.h9__btn:hover {
		background: rgba(255, 140, 30, 0.25);
		box-shadow: 0 0 60px rgba(255, 140, 30, 0.35);
		transform: translateY(-2px);
	}

	.h9__data {
		display: flex;
		gap: 0.5rem;
		font-family: var(--font-mono);
		font-size: 0.7rem;
		color: rgba(255, 180, 80, 0.35);
		letter-spacing: 0.06em;
	}

	.h9__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at center 42%, transparent 22%, rgba(0,0,4,0.8) 100%);
		pointer-events: none;
		z-index: 5;
	}
</style>
