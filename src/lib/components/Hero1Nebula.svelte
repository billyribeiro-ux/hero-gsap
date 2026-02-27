<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { ArrowRight, Planet } from 'phosphor-svelte';

	let section: HTMLElement;
	let canvas: HTMLCanvasElement;
	let titleRef: HTMLElement;
	let subtitleRef: HTMLElement;
	let ctaRef: HTMLElement;
	let badgeRef: HTMLElement;
	let overlayRef: HTMLElement;

	type Star = {
		x: number; y: number; z: number; pz: number;
		vx: number; vy: number; vz: number;
		r: number; opacity: number; trail: { x: number; y: number }[];
	};

	onMount(() => {
		const ctx = gsap.context(() => {}, section);
		const c = canvas.getContext('2d')!;
		let W = canvas.width = window.innerWidth;
		let H = canvas.height = window.innerHeight;
		let raf: number;
		let warpSpeed = 0;
		let targetWarp = 0.4;

		const STAR_COUNT = 600;
		const stars: Star[] = Array.from({ length: STAR_COUNT }, () => makestar(W, H));

		function makestar(w: number, h: number): Star {
			return {
				x: (Math.random() - 0.5) * w * 2,
				y: (Math.random() - 0.5) * h * 2,
				z: Math.random() * w,
				pz: 0,
				vx: 0, vy: 0, vz: 0,
				r: Math.random() * 1.5 + 0.3,
				opacity: Math.random(),
				trail: [],
			};
		}

		function drawStars() {
			c.fillStyle = 'rgba(0,0,4,0.18)';
			c.fillRect(0, 0, W, H);

			const cx = W / 2, cy = H / 2;
			warpSpeed += (targetWarp - warpSpeed) * 0.02;

			for (const s of stars) {
				s.pz = s.z;
				s.z -= 4 + warpSpeed * 14;
				if (s.z <= 0) {
					s.x = (Math.random() - 0.5) * W * 2;
					s.y = (Math.random() - 0.5) * H * 2;
					s.z = W;
					s.pz = s.z;
					s.trail = [];
				}
				const sx = (s.x / s.z) * W + cx;
				const sy = (s.y / s.z) * H + cy;
				const px = (s.x / s.pz) * W + cx;
				const py = (s.y / s.pz) * H + cy;
				const size = Math.max(0.1, (1 - s.z / W) * 3.2 + s.r * 0.4);
				const brightness = Math.min(1, (1 - s.z / W) * 2);
				const trailLen = warpSpeed * 18;
				const dx = sx - px, dy = sy - py;
				const dist = Math.sqrt(dx * dx + dy * dy);

				if (dist > 0.5 && trailLen > 0.5) {
					const g = c.createLinearGradient(px, py, sx, sy);
					g.addColorStop(0, `rgba(160,180,255,0)`);
					g.addColorStop(1, `rgba(200,220,255,${brightness * 0.9})`);
					c.beginPath();
					c.strokeStyle = g;
					c.lineWidth = size * 0.7;
					c.moveTo(px, py);
					c.lineTo(sx, sy);
					c.stroke();
				}

				c.beginPath();
				c.arc(sx, sy, size, 0, Math.PI * 2);

				const hue = 200 + Math.sin(s.x * 0.01) * 40;
				c.fillStyle = `hsla(${hue},80%,85%,${brightness})`;
				c.fill();

				if (size > 1.5 && brightness > 0.7) {
					c.beginPath();
					c.arc(sx, sy, size * 3.5, 0, Math.PI * 2);
					const glow = c.createRadialGradient(sx, sy, 0, sx, sy, size * 3.5);
					glow.addColorStop(0, `hsla(${hue},100%,90%,${brightness * 0.25})`);
					glow.addColorStop(1, 'transparent');
					c.fillStyle = glow;
					c.fill();
				}
			}

			const nebulaColors = [
				[80, 0, 160, 0.025],
				[0, 100, 200, 0.02],
				[160, 0, 80, 0.018],
			];
			for (const [r, g, b, a] of nebulaColors) {
				const ng = c.createRadialGradient(
					cx + Math.sin(Date.now() * 0.0003) * 120,
					cy + Math.cos(Date.now() * 0.0002) * 80,
					10,
					cx, cy, W * 0.55
				);
				ng.addColorStop(0, `rgba(${r},${g},${b},${a * 4})`);
				ng.addColorStop(1, `rgba(${r},${g},${b},0)`);
				c.fillStyle = ng;
				c.fillRect(0, 0, W, H);
			}

			raf = requestAnimationFrame(drawStars);
		}

		raf = requestAnimationFrame(drawStars);

		const onResize = () => {
			W = canvas.width = window.innerWidth;
			H = canvas.height = window.innerHeight;
		};
		window.addEventListener('resize', onResize);

		gsap.set([badgeRef, titleRef, subtitleRef, ctaRef], { autoAlpha: 0, y: 48 });
		gsap.set(overlayRef, { scaleX: 1, transformOrigin: 'left center' });

		const tl = gsap.timeline({ delay: 0.3 });
		tl.to(overlayRef, { scaleX: 0, duration: 1.2, ease: 'expo.inOut' })
			.to(badgeRef, { autoAlpha: 1, y: 0, duration: 0.8, ease: 'power3.out' }, '-=0.3')
			.to(titleRef, { autoAlpha: 1, y: 0, duration: 1, ease: 'power3.out' }, '-=0.55')
			.to(subtitleRef, { autoAlpha: 1, y: 0, duration: 0.9, ease: 'power3.out' }, '-=0.6')
			.to(ctaRef, { autoAlpha: 1, y: 0, duration: 0.8, ease: 'power3.out' }, '-=0.5');

		gsap.to(titleRef, {
			textShadow: '0 0 80px rgba(120,100,255,0.8), 0 0 160px rgba(80,120,255,0.4)',
			duration: 2.5,
			repeat: -1,
			yoyo: true,
			ease: 'sine.inOut',
			delay: 2,
		});

		const mousemove = (e: MouseEvent) => {
			const rx = (e.clientY / H - 0.5) * 6;
			const ry = (e.clientX / W - 0.5) * -6;
			gsap.to(titleRef, { rotateX: rx, rotateY: ry, duration: 1.2, ease: 'power2.out', transformPerspective: 800 });
		};
		section.addEventListener('mousemove', mousemove);

		return () => {
			cancelAnimationFrame(raf);
			window.removeEventListener('resize', onResize);
			section.removeEventListener('mousemove', mousemove);
			ctx.revert();
		};
	});
</script>

<section class="h1" bind:this={section}>
	<canvas bind:this={canvas}></canvas>
	<div class="h1__overlay" bind:this={overlayRef}></div>

	<div class="h1__content">
		<div class="h1__badge" bind:this={badgeRef}>
			<Planet size={16} weight="fill" />
			<span>Sector 7 · Deep Space Initiative</span>
		</div>

		<h1 class="h1__title" bind:this={titleRef}>
			BEYOND<br />
			<span class="h1__title-accent">THE VOID</span>
		</h1>

		<p class="h1__subtitle" bind:this={subtitleRef}>
			A journey through 13 billion light-years of darkness.<br />
			Where gravity bends light — and time surrenders.
		</p>

		<div class="h1__cta" bind:this={ctaRef}>
			<button class="btn-primary">
				<span>Enter the Universe</span>
				<ArrowRight size={18} weight="bold" />
			</button>
			<button class="btn-ghost">Watch Trailer</button>
		</div>
	</div>

	<div class="h1__vignette"></div>
	<div class="h1__scanlines"></div>
</section>

<style>
	.h1 {
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
	}

	.h1__overlay {
		position: absolute;
		inset: 0;
		background: #000;
		z-index: 10;
		transform-origin: left center;
	}

	.h1__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at center, transparent 40%, rgba(0,0,4,0.85) 100%);
		pointer-events: none;
		z-index: 2;
	}

	.h1__scanlines {
		position: absolute;
		inset: 0;
		background: repeating-linear-gradient(
			0deg,
			transparent,
			transparent 2px,
			rgba(0,0,0,0.03) 2px,
			rgba(0,0,0,0.03) 4px
		);
		pointer-events: none;
		z-index: 3;
	}

	.h1__content {
		position: relative;
		z-index: 5;
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1.75rem;
	}

	.h1__badge {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0.4rem 1rem;
		border: 1px solid rgba(120, 100, 255, 0.35);
		border-radius: 99px;
		background: rgba(80, 60, 200, 0.12);
		color: rgba(180, 170, 255, 0.9);
		font-size: 0.72rem;
		font-family: var(--font-mono);
		letter-spacing: 0.12em;
		text-transform: uppercase;
		backdrop-filter: blur(8px);
	}

	.h1__title {
		font-family: var(--font-cinematic);
		font-size: clamp(5rem, 14vw, 14rem);
		line-height: 0.9;
		letter-spacing: 0.06em;
		color: #fff;
		text-shadow: 0 0 40px rgba(120, 100, 255, 0.5), 0 0 100px rgba(80, 120, 255, 0.2);
		transform-style: preserve-3d;
	}

	.h1__title-accent {
		display: block;
		background: linear-gradient(135deg, #a78bfa 0%, #60a5fa 50%, #a78bfa 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
		background-size: 200% 100%;
		animation: shimmer 4s linear infinite;
	}

	@keyframes shimmer {
		0% { background-position: 200% center; }
		100% { background-position: -200% center; }
	}

	.h1__subtitle {
		font-size: clamp(0.9rem, 1.5vw, 1.15rem);
		color: rgba(180, 185, 220, 0.75);
		line-height: 1.75;
		max-width: 540px;
		font-weight: 300;
		letter-spacing: 0.02em;
	}

	.h1__cta {
		display: flex;
		gap: 1rem;
		margin-top: 0.5rem;
	}

	.btn-primary {
		display: inline-flex;
		align-items: center;
		gap: 0.55rem;
		padding: 0.85rem 2rem;
		background: linear-gradient(135deg, #6366f1, #8b5cf6);
		color: #fff;
		font-family: var(--font-sans);
		font-size: 0.9rem;
		font-weight: 600;
		border: none;
		border-radius: 8px;
		cursor: pointer;
		box-shadow: 0 0 40px rgba(99, 102, 241, 0.45), 0 4px 20px rgba(0,0,0,0.4);
		transition: transform 0.2s, box-shadow 0.2s;
	}

	.btn-primary:hover {
		transform: translateY(-2px) scale(1.02);
		box-shadow: 0 0 60px rgba(99, 102, 241, 0.7), 0 8px 30px rgba(0,0,0,0.5);
	}

	.btn-ghost {
		display: inline-flex;
		align-items: center;
		padding: 0.85rem 2rem;
		background: rgba(255,255,255,0.05);
		color: rgba(255,255,255,0.75);
		font-family: var(--font-sans);
		font-size: 0.9rem;
		font-weight: 500;
		border: 1px solid rgba(255,255,255,0.12);
		border-radius: 8px;
		cursor: pointer;
		backdrop-filter: blur(8px);
		transition: background 0.2s, color 0.2s;
	}

	.btn-ghost:hover {
		background: rgba(255,255,255,0.1);
		color: #fff;
	}
</style>
