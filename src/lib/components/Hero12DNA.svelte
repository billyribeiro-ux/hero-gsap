<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { Dna, ArrowRight, Atom } from 'phosphor-svelte';

	let section: HTMLElement;
	let canvas: HTMLCanvasElement;
	let titleRef: HTMLElement;
	let subtitleRef: HTMLElement;
	let ctaRef: HTMLElement;
	let eyebrowRef: HTMLElement;
	let dataReadoutRef: HTMLElement;

	const SEQUENCE = 'ATCGATCGGCTATCGATCGGCTATCG';
	const CODONS = ['AUG', 'GCU', 'UAA', 'CCG', 'UGC', 'GAU', 'AAA', 'GGU'];

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let W = (canvas.width = window.innerWidth);
		let H = (canvas.height = window.innerHeight);
		let raf: number;
		let time = 0;
		let codonIdx = 0;

		const codonInterval = setInterval(() => {
			codonIdx = (codonIdx + 1) % CODONS.length;
			if (dataReadoutRef) {
				const el = dataReadoutRef.querySelector('.codon-val');
				if (el) {
					gsap.fromTo(el, { opacity: 0, y: -8 }, { opacity: 1, y: 0, duration: 0.3, ease: 'power2.out' });
					el.textContent = CODONS[codonIdx];
				}
			}
		}, 600);

		// Helix parameters
		const PAIRS = 28;
		const HELIX_H = H * 1.1;
		const HELIX_W = 120;
		const cx = W * 0.72;
		const cy = H * 0.5;

		// Floating code characters
		type FloatChar = { x: number; y: number; char: string; alpha: number; speed: number; size: number };
		const floatChars: FloatChar[] = Array.from({ length: 60 }, () => ({
			x: W * 0.4 + Math.random() * W * 0.55,
			y: Math.random() * H,
			char: 'ATCG'[Math.floor(Math.random() * 4)],
			alpha: 0.05 + Math.random() * 0.2,
			speed: 0.3 + Math.random() * 0.8,
			size: 10 + Math.random() * 14,
		}));

		// Particle connections
		type ConnParticle = { x: number; y: number; vx: number; vy: number; alpha: number; size: number; hue: number };
		const connParticles: ConnParticle[] = Array.from({ length: 80 }, () => ({
			x: cx + (Math.random() - 0.5) * 400,
			y: Math.random() * H,
			vx: (Math.random() - 0.5) * 0.4,
			vy: (Math.random() - 0.5) * 0.4,
			alpha: 0.1 + Math.random() * 0.3,
			size: 1 + Math.random() * 2,
			hue: 160 + Math.random() * 60,
		}));

		function drawHelix() {
			const scroll = time * 0.4;
			const points: { x: number; y: number; z: number; pair: number; strand: number }[] = [];

			for (let i = 0; i < PAIRS; i++) {
				const t = (i / PAIRS) * Math.PI * 4 + scroll;
				const y = cy - HELIX_H / 2 + (i / PAIRS) * HELIX_H;

				// Strand A
				const axA = cx + Math.cos(t) * HELIX_W;
				const zA = Math.sin(t);
				// Strand B (180° offset)
				const axB = cx + Math.cos(t + Math.PI) * HELIX_W;
				const zB = Math.sin(t + Math.PI);

				points.push({ x: axA, y, z: zA, pair: i, strand: 0 });
				points.push({ x: axB, y, z: zB, pair: i, strand: 1 });
			}

			// Sort by z for depth
			const strandA = points.filter(p => p.strand === 0).sort((a, b) => a.pair - b.pair);
			const strandB = points.filter(p => p.strand === 1).sort((a, b) => a.pair - b.pair);

			// Draw backbone strands
			for (const strand of [strandA, strandB]) {
				const hue = strand === strandA ? 180 : 280;
				ctx.beginPath();
				for (let i = 0; i < strand.length; i++) {
					const p = strand[i];
					if (p.y < -20 || p.y > H + 20) continue;
					const depth = (p.z + 1) / 2;
					ctx.globalAlpha = 0.3 + depth * 0.55;
					const r = 2.5 + depth * 3;
					if (i === 0 || strand[i-1].y < -20 || strand[i-1].y > H + 20) {
						ctx.beginPath();
					}
					// Glow node
					const glow = ctx.createRadialGradient(p.x, p.y, 0, p.x, p.y, r * 3.5);
					glow.addColorStop(0, `hsla(${hue},90%,70%,${depth * 0.8})`);
					glow.addColorStop(1, 'hsla(0,0%,0%,0)');
					ctx.fillStyle = glow;
					ctx.beginPath();
					ctx.arc(p.x, p.y, r * 3, 0, Math.PI * 2);
					ctx.fill();

					ctx.fillStyle = `hsla(${hue},80%,75%,1)`;
					ctx.shadowColor = `hsla(${hue},90%,80%,1)`;
					ctx.shadowBlur = 8 + depth * 12;
					ctx.beginPath();
					ctx.arc(p.x, p.y, r, 0, Math.PI * 2);
					ctx.fill();
					ctx.shadowBlur = 0;
				}
			}

			// Draw base pair rungs
			for (let i = 0; i < strandA.length; i++) {
				const a = strandA[i];
				const b = strandB[i];
				if (a.y < -20 || a.y > H + 20) continue;
				const depth = (a.z + 1) / 2;
				const baseChar = SEQUENCE[i % SEQUENCE.length];
				const hue = baseChar === 'A' ? 120 : baseChar === 'T' ? 200 : baseChar === 'C' ? 280 : 40;

				// Rung line
				const grad = ctx.createLinearGradient(a.x, a.y, b.x, b.y);
				grad.addColorStop(0, `hsla(180,80%,65%,${depth * 0.6})`);
				grad.addColorStop(0.5, `hsla(${hue},90%,75%,${depth * 0.8})`);
				grad.addColorStop(1, `hsla(280,80%,65%,${depth * 0.6})`);
				ctx.beginPath();
				ctx.moveTo(a.x, a.y);
				ctx.lineTo(b.x, b.y);
				ctx.strokeStyle = grad;
				ctx.lineWidth = 1.5 + depth * 1.5;
				ctx.globalAlpha = 0.5 + depth * 0.4;
				ctx.stroke();

				// Base pair label
				if (depth > 0.4 && i % 3 === 0) {
					const mx = (a.x + b.x) / 2;
					const my = (a.y + b.y) / 2;
					ctx.globalAlpha = depth * 0.7;
					ctx.fillStyle = `hsla(${hue},90%,80%,1)`;
					ctx.font = `bold ${9 + depth * 4}px var(--font-mono)`;
					ctx.textAlign = 'center';
					ctx.textBaseline = 'middle';
					ctx.fillText(baseChar, mx, my);
				}
			}

			ctx.globalAlpha = 1;
			ctx.shadowBlur = 0;
		}

		function draw() {
			ctx.fillStyle = 'rgba(2,8,16,0.2)';
			ctx.fillRect(0, 0, W, H);
			time += 0.016;

			// Floating ATCG chars
			for (const fc of floatChars) {
				fc.y -= fc.speed;
				if (fc.y < -20) { fc.y = H + 20; fc.x = W * 0.4 + Math.random() * W * 0.55; }
				ctx.globalAlpha = fc.alpha * (0.5 + Math.sin(time + fc.x) * 0.5);
				ctx.fillStyle = '#4ade80';
				ctx.font = `${fc.size}px var(--font-mono)`;
				ctx.textAlign = 'left';
				ctx.fillText(fc.char, fc.x, fc.y);
			}

			// Connection particles near helix
			for (let i = 0; i < connParticles.length; i++) {
				const p = connParticles[i];
				p.x += p.vx;
				p.y += p.vy;
				if (p.x < W * 0.35 || p.x > W + 20) { p.x = W * 0.4 + Math.random() * W * 0.5; }
				if (p.y < 0 || p.y > H) { p.y = Math.random() * H; }

				// Connect nearby particles
				for (let j = i + 1; j < connParticles.length; j++) {
					const q = connParticles[j];
					const dx = p.x - q.x, dy = p.y - q.y;
					const dist = Math.sqrt(dx*dx + dy*dy);
					if (dist < 80) {
						ctx.globalAlpha = (1 - dist / 80) * p.alpha * 0.4;
						ctx.strokeStyle = `hsla(${p.hue},80%,65%,1)`;
						ctx.lineWidth = 0.5;
						ctx.beginPath();
						ctx.moveTo(p.x, p.y);
						ctx.lineTo(q.x, q.y);
						ctx.stroke();
					}
				}

				ctx.globalAlpha = p.alpha;
				ctx.fillStyle = `hsla(${p.hue},80%,70%,1)`;
				ctx.shadowColor = `hsla(${p.hue},90%,70%,0.8)`;
				ctx.shadowBlur = 4;
				ctx.beginPath();
				ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
				ctx.fill();
			}
			ctx.shadowBlur = 0;

			drawHelix();

			raf = requestAnimationFrame(draw);
		}
		draw();

		// Entrance
		gsap.set([eyebrowRef, titleRef, subtitleRef, ctaRef, dataReadoutRef], { autoAlpha: 0, x: -40 });
		const tl = gsap.timeline({ delay: 0.3 });
		tl.to(eyebrowRef, { autoAlpha: 1, x: 0, duration: 0.7, ease: 'power3.out' })
			.to(titleRef, { autoAlpha: 1, x: 0, duration: 1.0, ease: 'power3.out' }, '-=0.3')
			.to(subtitleRef, { autoAlpha: 1, x: 0, duration: 0.8, ease: 'power3.out' }, '-=0.5')
			.to(dataReadoutRef, { autoAlpha: 1, x: 0, duration: 0.7, ease: 'power3.out' }, '-=0.45')
			.to(ctaRef, { autoAlpha: 1, x: 0, duration: 0.7, ease: 'power3.out' }, '-=0.4');

		// Title breathe
		gsap.to(titleRef, {
			textShadow: '0 0 60px rgba(74,222,128,0.6), 0 0 120px rgba(74,222,128,0.25)',
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
			clearInterval(codonInterval);
			window.removeEventListener('resize', onResize);
		};
	});
</script>

<section class="h12" bind:this={section}>
	<canvas bind:this={canvas}></canvas>
	<div class="h12__bg"></div>

	<div class="h12__content">
		<div class="h12__eyebrow" bind:this={eyebrowRef}>
			<Atom size={13} weight="fill" />
			<span>PROJECT GENESIS · CLASSIFIED LEVEL 5</span>
		</div>

		<h1 class="h12__title" bind:this={titleRef}>
			<span class="h12__t1">REWRITE</span>
			<span class="h12__t2">LIFE</span>
			<span class="h12__t3">ITSELF</span>
		</h1>

		<p class="h12__subtitle" bind:this={subtitleRef}>
			3 billion base pairs.<br />
			One sequence changes everything.
		</p>

		<div class="h12__data-readout" bind:this={dataReadoutRef}>
			<div class="h12__data-row">
				<span class="h12__data-key">CODON</span>
				<span class="codon-val h12__data-val">AUG</span>
			</div>
			<div class="h12__data-row">
				<span class="h12__data-key">STRAND</span>
				<span class="h12__data-val">5'→3'</span>
			</div>
			<div class="h12__data-row">
				<span class="h12__data-key">EXPR</span>
				<span class="h12__data-val h12__data-active">ACTIVE</span>
			</div>
		</div>

		<div class="h12__cta" bind:this={ctaRef}>
			<button class="h12__btn">
				<Dna size={16} weight="fill" />
				<span>Sequence Now</span>
				<ArrowRight size={15} weight="bold" />
			</button>
		</div>
	</div>

	<div class="h12__vignette"></div>
</section>

<style>
	.h12 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #020810;
		display: flex;
		align-items: center;
		padding: 0 8vw;
	}

	canvas {
		position: absolute;
		inset: 0;
		width: 100%; height: 100%;
		z-index: 2;
		pointer-events: none;
	}

	.h12__bg {
		position: absolute;
		inset: 0;
		background:
			radial-gradient(ellipse at 25% 50%, rgba(20, 80, 40, 0.12) 0%, transparent 55%),
			radial-gradient(ellipse at 70% 50%, rgba(40, 100, 160, 0.08) 0%, transparent 55%);
		z-index: 1;
	}

	.h12__content {
		position: relative;
		z-index: 10;
		display: flex;
		flex-direction: column;
		gap: 1.6rem;
		max-width: 600px;
	}

	.h12__eyebrow {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		font-family: var(--font-mono);
		font-size: 0.66rem;
		letter-spacing: 0.18em;
		text-transform: uppercase;
		color: rgba(74, 222, 128, 0.7);
		border-left: 2px solid rgba(74, 222, 128, 0.4);
		padding-left: 0.75rem;
	}

	.h12__title {
		display: flex;
		flex-direction: column;
		line-height: 0.88;
	}

	.h12__t1 {
		font-family: var(--font-cinematic);
		font-size: clamp(2.5rem, 5.5vw, 6.5rem);
		letter-spacing: 0.12em;
		color: rgba(100, 220, 140, 0.65);
		text-shadow: 0 0 30px rgba(74, 222, 128, 0.35);
	}

	.h12__t2 {
		font-family: var(--font-cinematic);
		font-size: clamp(6rem, 14vw, 16rem);
		letter-spacing: 0.03em;
		background: linear-gradient(135deg, #fff 0%, #86efac 30%, #4ade80 60%, #16a34a 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
		filter: drop-shadow(0 0 30px rgba(74,222,128,0.5));
		line-height: 0.85;
	}

	.h12__t3 {
		font-family: var(--font-cinematic);
		font-size: clamp(2.5rem, 5.5vw, 6.5rem);
		letter-spacing: 0.25em;
		color: transparent;
		-webkit-text-stroke: 1px rgba(74, 222, 128, 0.45);
	}

	.h12__subtitle {
		font-size: 0.93rem;
		color: rgba(140, 200, 160, 0.6);
		font-weight: 300;
		line-height: 1.8;
		letter-spacing: 0.03em;
	}

	.h12__data-readout {
		display: flex;
		flex-direction: column;
		gap: 0.4rem;
		padding: 0.9rem 1.1rem;
		border: 1px solid rgba(74, 222, 128, 0.15);
		background: rgba(74, 222, 128, 0.03);
		width: fit-content;
		backdrop-filter: blur(8px);
	}

	.h12__data-row {
		display: flex;
		gap: 1.2rem;
		align-items: center;
		font-family: var(--font-mono);
		font-size: 0.72rem;
	}

	.h12__data-key {
		color: rgba(74, 222, 128, 0.4);
		letter-spacing: 0.1em;
		width: 52px;
	}

	.h12__data-val {
		color: rgba(200, 240, 210, 0.85);
		letter-spacing: 0.08em;
	}

	.h12__data-active {
		color: #4ade80;
		text-shadow: 0 0 10px rgba(74, 222, 128, 0.6);
		animation: activePulse 1.2s ease-in-out infinite;
	}

	@keyframes activePulse {
		0%, 100% { opacity: 0.9; }
		50% { opacity: 0.4; }
	}

	.h12__cta {
		margin-top: 0.3rem;
	}

	.h12__btn {
		display: inline-flex;
		align-items: center;
		gap: 0.6rem;
		padding: 0.88rem 2.1rem;
		background: rgba(74, 222, 128, 0.1);
		border: 1px solid rgba(74, 222, 128, 0.4);
		color: rgba(180, 240, 200, 0.95);
		font-size: 0.88rem;
		font-weight: 600;
		letter-spacing: 0.07em;
		cursor: pointer;
		backdrop-filter: blur(10px);
		transition: all 0.25s;
		box-shadow: 0 0 25px rgba(74, 222, 128, 0.1);
	}

	.h12__btn:hover {
		background: rgba(74, 222, 128, 0.22);
		box-shadow: 0 0 50px rgba(74, 222, 128, 0.35);
		transform: translateY(-2px);
	}

	.h12__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at 30% 50%, transparent 35%, rgba(2,8,16,0.88) 100%);
		pointer-events: none;
		z-index: 5;
	}
</style>
