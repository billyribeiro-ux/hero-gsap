<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { Skull, ArrowRight } from 'phosphor-svelte';

	let section: HTMLElement;
	let canvas: HTMLCanvasElement;
	let titleRef: HTMLElement;
	let subtitleRef: HTMLElement;
	let ctaRef: HTMLElement;
	let overlayRef: HTMLElement;

	type Shard = {
		x: number; y: number; vx: number; vy: number;
		rot: number; vrot: number;
		w: number; h: number;
		alpha: number; color: string;
		gravity: number; friction: number;
		clipPath: string;
	};

	type Particle = {
		x: number; y: number; vx: number; vy: number;
		size: number; alpha: number; color: string; gravity: number;
	};

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let W = (canvas.width = window.innerWidth);
		let H = (canvas.height = window.innerHeight);
		let raf: number;
		let shards: Shard[] = [];
		let particles: Particle[] = [];
		let phase: 'idle' | 'explode' | 'settle' = 'idle';
		let shockwaveR = 0;
		let shockwaveAlpha = 0;

		const colors = ['#ff3a00', '#ff7700', '#ffcc00', '#ffffff', '#ff0055', '#cc00ff'];

		function makeShards(cx: number, cy: number, count = 120) {
			shards = [];
			for (let i = 0; i < count; i++) {
				const angle = Math.random() * Math.PI * 2;
				const speed = 4 + Math.random() * 22;
				const w = 10 + Math.random() * 80;
				const h = 4 + Math.random() * 40;
				const color = colors[Math.floor(Math.random() * colors.length)];
				const pts = generateShardPath(w, h);
				shards.push({
					x: cx + (Math.random() - 0.5) * 200,
					y: cy + (Math.random() - 0.5) * 100,
					vx: Math.cos(angle) * speed,
					vy: Math.sin(angle) * speed - Math.random() * 8,
					rot: Math.random() * Math.PI * 2,
					vrot: (Math.random() - 0.5) * 0.3,
					w, h,
					alpha: 0.8 + Math.random() * 0.2,
					color,
					gravity: 0.25 + Math.random() * 0.2,
					friction: 0.97 + Math.random() * 0.02,
					clipPath: pts,
				});
			}
		}

		function generateShardPath(w: number, h: number): string {
			const pts = 3 + Math.floor(Math.random() * 4);
			const verts: [number, number][] = [];
			for (let i = 0; i < pts; i++) {
				verts.push([(Math.random() - 0.5) * w, (Math.random() - 0.5) * h]);
			}
			return verts.map((v, i) => `${i === 0 ? 'M' : 'L'}${v[0]},${v[1]}`).join(' ') + 'Z';
		}

		function makeParticles(cx: number, cy: number, count = 300) {
			particles = [];
			for (let i = 0; i < count; i++) {
				const angle = Math.random() * Math.PI * 2;
				const speed = 2 + Math.random() * 18;
				particles.push({
					x: cx + (Math.random() - 0.5) * 160,
					y: cy + (Math.random() - 0.5) * 80,
					vx: Math.cos(angle) * speed,
					vy: Math.sin(angle) * speed - Math.random() * 6,
					size: 1 + Math.random() * 4,
					alpha: 1,
					color: colors[Math.floor(Math.random() * colors.length)],
					gravity: 0.1 + Math.random() * 0.2,
				});
			}
		}

		function drawScene() {
			ctx.clearRect(0, 0, W, H);

			if (phase === 'explode' || phase === 'settle') {
				// Shockwave ring
				if (shockwaveAlpha > 0) {
					ctx.save();
					ctx.beginPath();
					ctx.arc(W / 2, H * 0.42, shockwaveR, 0, Math.PI * 2);
					ctx.strokeStyle = `rgba(255,180,50,${shockwaveAlpha})`;
					ctx.lineWidth = 6;
					ctx.stroke();
					ctx.restore();
					shockwaveR += 28;
					shockwaveAlpha -= 0.025;
				}

				// Ember glow at epicenter
				const eg = ctx.createRadialGradient(W / 2, H * 0.42, 0, W / 2, H * 0.42, 180);
				const firstAlpha = shards[0] ? shards[0].alpha * 0.1 : 0;
			eg.addColorStop(0, `rgba(255,120,0,${Math.max(0, 0.35 - firstAlpha)})`);
				eg.addColorStop(1, 'rgba(0,0,0,0)');
				ctx.fillStyle = eg;
				ctx.fillRect(0, 0, W, H);

				// Draw shards
				for (const s of shards) {
					ctx.save();
					ctx.translate(s.x, s.y);
					ctx.rotate(s.rot);
					ctx.globalAlpha = s.alpha;
					ctx.fillStyle = s.color;
					ctx.shadowColor = s.color;
					ctx.shadowBlur = 12;
					const path = new Path2D(s.clipPath);
					ctx.fill(path);
					ctx.restore();
					// physics
					s.vy += s.gravity;
					s.vx *= s.friction;
					s.vy *= s.friction;
					s.x += s.vx;
					s.y += s.vy;
					s.rot += s.vrot;
					s.alpha -= 0.004;
					if (s.alpha < 0) s.alpha = 0;
				}

				// Draw particles
				for (const p of particles) {
					ctx.save();
					ctx.globalAlpha = p.alpha;
					ctx.fillStyle = p.color;
					ctx.shadowColor = p.color;
					ctx.shadowBlur = 8;
					ctx.beginPath();
					ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
					ctx.fill();
					ctx.restore();
					p.vy += p.gravity;
					p.vx *= 0.98;
					p.vy *= 0.98;
					p.x += p.vx;
					p.y += p.vy;
					p.alpha -= 0.008;
					if (p.alpha < 0) p.alpha = 0;
				}
			}

			raf = requestAnimationFrame(drawScene);
		}

		drawScene();

		// GSAP entrance
		gsap.set([titleRef, subtitleRef, ctaRef], { autoAlpha: 0 });
		gsap.set(overlayRef, { scaleX: 1, transformOrigin: 'left' });

		const tl = gsap.timeline({ delay: 0.2 });
		tl.to(overlayRef, { scaleX: 0, duration: 1.0, ease: 'expo.inOut' })
			.to(titleRef, {
				autoAlpha: 1, duration: 0.01,
				onComplete: () => {
					gsap.fromTo(titleRef,
						{ scale: 0.4, filter: 'blur(30px)', autoAlpha: 0 },
						{ scale: 1, filter: 'blur(0px)', autoAlpha: 1, duration: 0.7, ease: 'expo.out' }
					);
				}
			}, '-=0.1')
			.to(subtitleRef, { autoAlpha: 1, y: 0, duration: 0.8, ease: 'power3.out' }, '+=0.3')
			.to(ctaRef, { autoAlpha: 1, duration: 0.7, ease: 'power2.out' }, '-=0.4');

		// Big shatter explosion
		gsap.delayedCall(1.6, () => {
			phase = 'explode';
			shockwaveR = 0;
			shockwaveAlpha = 1;
			makeShards(W / 2, H * 0.42);
			makeParticles(W / 2, H * 0.42);

			gsap.fromTo(section,
				{ filter: 'brightness(3) saturate(0)' },
				{ filter: 'brightness(1) saturate(1)', duration: 0.6, ease: 'power3.out' }
			);
			gsap.to(titleRef, {
				scale: 1.08, duration: 0.12, ease: 'power4.out', yoyo: true, repeat: 1,
				onComplete: () => { gsap.set(titleRef, { scale: 1 }); }
			});
			phase = 'settle';
		});

		// Repeating mini-explosions
		let microInterval: ReturnType<typeof setInterval>;
		gsap.delayedCall(2.2, () => {
			microInterval = setInterval(() => {
				const x = W / 2 + (Math.random() - 0.5) * 400;
				const y = H * 0.35 + (Math.random() - 0.5) * 200;
				makeParticles(x, y, 40);
			}, 1800);
		});

		const onResize = () => {
			W = canvas.width = window.innerWidth;
			H = canvas.height = window.innerHeight;
		};
		window.addEventListener('resize', onResize);

		return () => {
			cancelAnimationFrame(raf);
			clearInterval(microInterval);
			window.removeEventListener('resize', onResize);
		};
	});
</script>

<section class="h7" bind:this={section}>
	<div class="h7__overlay" bind:this={overlayRef}></div>
	<canvas bind:this={canvas}></canvas>

	<div class="h7__bg">
		<div class="h7__bg-grid"></div>
		<div class="h7__bg-radial"></div>
	</div>

	<div class="h7__content">
		<div class="h7__eyebrow">
			<Skull size={14} weight="fill" />
			<span>SUMMER 2025 · EVENT FILM</span>
		</div>

		<h1 class="h7__title" bind:this={titleRef}>
			<span class="h7__title-line h7__t1">OBLITERATE</span>
			<span class="h7__title-line h7__t2">EVERYTHING</span>
		</h1>

		<p class="h7__subtitle" bind:this={subtitleRef}>
			When the world ends, only the fire remains.
		</p>

		<div class="h7__cta" bind:this={ctaRef}>
			<button class="h7__btn">
				<span>Witness the Destruction</span>
				<ArrowRight size={17} weight="bold" />
			</button>
			<div class="h7__tag">IMAX · RATED R</div>
		</div>
	</div>

	<div class="h7__vignette"></div>
</section>

<style>
	.h7 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #08030a;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.h7__overlay {
		position: absolute;
		inset: 0;
		background: #08030a;
		z-index: 30;
	}

	canvas {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		z-index: 3;
		pointer-events: none;
		mix-blend-mode: screen;
	}

	.h7__bg {
		position: absolute;
		inset: 0;
		z-index: 1;
	}

	.h7__bg-grid {
		position: absolute;
		inset: 0;
		background-image:
			linear-gradient(rgba(255, 60, 0, 0.05) 1px, transparent 1px),
			linear-gradient(90deg, rgba(255, 60, 0, 0.05) 1px, transparent 1px);
		background-size: 80px 80px;
	}

	.h7__bg-radial {
		position: absolute;
		inset: 0;
		background:
			radial-gradient(ellipse at 50% 42%, rgba(180, 40, 0, 0.22) 0%, transparent 55%),
			radial-gradient(ellipse at 50% 42%, rgba(255, 100, 0, 0.1) 0%, transparent 30%);
	}

	.h7__content {
		position: relative;
		z-index: 10;
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1.8rem;
	}

	.h7__eyebrow {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		font-family: var(--font-mono);
		font-size: 0.68rem;
		letter-spacing: 0.2em;
		text-transform: uppercase;
		color: rgba(255, 100, 30, 0.8);
		border: 1px solid rgba(255, 80, 0, 0.3);
		padding: 0.3rem 0.9rem;
	}

	.h7__title {
		display: flex;
		flex-direction: column;
		align-items: center;
		line-height: 0.88;
		gap: 0;
	}

	.h7__title-line {
		display: block;
		font-family: var(--font-cinematic);
		letter-spacing: 0.05em;
	}

	.h7__t1 {
		font-size: clamp(3.5rem, 9vw, 10rem);
		color: #fff;
		text-shadow:
			0 0 20px rgba(255, 100, 0, 0.6),
			0 0 60px rgba(255, 60, 0, 0.3),
			0 0 120px rgba(200, 0, 0, 0.2);
	}

	.h7__t2 {
		font-size: clamp(5rem, 14vw, 16rem);
		background: linear-gradient(160deg, #fff 0%, #ff8c00 35%, #ff2200 65%, #cc0000 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
		filter: drop-shadow(0 0 30px rgba(255, 80, 0, 0.7));
	}

	.h7__subtitle {
		font-size: 1rem;
		color: rgba(255, 180, 100, 0.65);
		font-weight: 300;
		letter-spacing: 0.1em;
		text-transform: uppercase;
	}

	.h7__cta {
		display: flex;
		align-items: center;
		gap: 1.2rem;
	}

	.h7__btn {
		display: inline-flex;
		align-items: center;
		gap: 0.6rem;
		padding: 0.9rem 2.2rem;
		background: linear-gradient(135deg, rgba(255, 80, 0, 0.25), rgba(180, 0, 0, 0.25));
		border: 1px solid rgba(255, 80, 0, 0.55);
		color: rgba(255, 200, 130, 0.95);
		font-size: 0.9rem;
		font-weight: 600;
		letter-spacing: 0.06em;
		cursor: pointer;
		backdrop-filter: blur(8px);
		transition: all 0.25s;
		box-shadow: 0 0 30px rgba(255, 80, 0, 0.25), inset 0 1px 0 rgba(255,255,255,0.08);
	}

	.h7__btn:hover {
		background: linear-gradient(135deg, rgba(255, 80, 0, 0.45), rgba(180, 0, 0, 0.45));
		box-shadow: 0 0 60px rgba(255, 80, 0, 0.55);
		transform: translateY(-2px);
	}

	.h7__tag {
		font-family: var(--font-mono);
		font-size: 0.68rem;
		letter-spacing: 0.15em;
		color: rgba(255, 100, 30, 0.5);
		border: 1px solid rgba(255, 100, 30, 0.2);
		padding: 0.3rem 0.7rem;
	}

	.h7__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at center, transparent 30%, rgba(8, 3, 10, 0.88) 100%);
		pointer-events: none;
		z-index: 4;
	}
</style>
