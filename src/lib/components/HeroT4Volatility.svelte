<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';

	let section: HTMLElement;
	let canvas: HTMLCanvasElement;
	let titleRef: HTMLElement;
	let subtitleRef: HTMLElement;
	let ctaRef: HTMLElement;
	let statsRef: HTMLElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let W = (canvas.width = window.innerWidth);
		let H = (canvas.height = window.innerHeight);
		let raf: number;
		let time = 0;

		// VIX-style wave surface
		const COLS = 60;
		const ROWS = 30;
		type SurfacePoint = { x: number; y: number; z: number; baseZ: number };
		const surface: SurfacePoint[][] = [];

		for (let r = 0; r < ROWS; r++) {
			surface.push([]);
			for (let c = 0; c < COLS; c++) {
				const bz = Math.sin(c * 0.3) * 0.5 + Math.sin(r * 0.4) * 0.5;
				surface[r].push({ x: 0, y: 0, z: bz, baseZ: bz });
			}
		}

		// Options chain data
		const STRIKES = [36000, 37000, 38000, 39000, 40000, 41000, 42000, 43000, 44000, 45000, 46000, 47000, 48000];
		const EXPIRIES = ['1D', '3D', '1W', '2W', '1M', '3M'];

		type OptionsCell = { strike: number; expiry: string; iv: number; delta: number; gamma: number; oi: number };
		const chain: OptionsCell[] = [];

		EXPIRIES.forEach((exp, ei) => {
			STRIKES.forEach((strike, si) => {
				const moneyness = (strike - 42000) / 42000;
				const iv = 0.45 + Math.abs(moneyness) * 2.2 + ei * 0.06 + Math.random() * 0.08;
				const delta = 0.5 - moneyness * 2;
				const gamma = 0.8 * Math.exp(-moneyness * moneyness * 30);
				const oi = Math.floor((500 - Math.abs(si - 6) * 40) * (1 + Math.random() * 0.3));
				chain.push({ strike, expiry: exp, iv, delta, gamma, oi });
			});
		});

		// IV surface animation
		let vixVal = 24.8;
		const vixInterval = setInterval(() => {
			vixVal += (Math.random() - 0.49) * 0.6;
			vixVal = Math.max(12, Math.min(80, vixVal));
			// Ripple effect on surface when VIX spikes
			if (Math.random() > 0.88) {
				const spikeMag = (Math.random() - 0.4) * 3;
				surface.forEach(row => row.forEach(pt => {
					pt.baseZ += spikeMag * Math.random() * 0.3;
					pt.baseZ = Math.max(-2, Math.min(2, pt.baseZ));
				}));
			}
		}, 400);

		function project3D(cx: number, cy: number, cz: number): [number, number] {
			// Isometric-style projection
			const tilt = 0.35;
			const scale = Math.min(W, H) * 0.022;
			const ox = W * 0.5;
			const oy = H * 0.52;
			const px = (cx - cy) * scale * 0.7 + ox;
			const py = ((cx + cy) * tilt - cz * 1.8) * scale + oy;
			return [px, py];
		}

		function drawSurface() {
			// Update surface z values
			surface.forEach((row, r) => {
				row.forEach((pt, c) => {
					pt.z = pt.baseZ
						+ Math.sin(time * 1.2 + c * 0.4) * 0.4
						+ Math.cos(time * 0.8 + r * 0.35) * 0.3
						+ Math.sin(time * 2.5 + c * 0.2 + r * 0.3) * 0.15;
				});
			});

			// Draw from back to front for depth
			for (let r = ROWS - 1; r >= 0; r--) {
				for (let c = COLS - 1; c >= 0; c--) {
					const cx = c - COLS / 2;
					const cy = r - ROWS / 2;
					const cz = surface[r][c].z;

					const [px, py] = project3D(cx, cy, cz);
					const [px1, py1] = r < ROWS - 1 ? project3D(cx, cy + 1, surface[r + 1]?.[c]?.z ?? cz) : [px, py];
					const [px2, py2] = c < COLS - 1 ? project3D(cx + 1, cy, surface[r]?.[c + 1]?.z ?? cz) : [px, py];
					const [px3, py3] = (r < ROWS - 1 && c < COLS - 1)
						? project3D(cx + 1, cy + 1, surface[r + 1]?.[c + 1]?.z ?? cz)
						: [px, py];

					// Color by z height (IV surface = volatility smile)
					const normalZ = (cz + 2) / 4;
					const hue = 200 - normalZ * 200; // blue=low vol, red=high vol
					const sat = 70 + normalZ * 30;
					const lum = 35 + normalZ * 25;
					const alpha = 0.45 + normalZ * 0.35;

					ctx.beginPath();
					ctx.moveTo(px, py);
					ctx.lineTo(px2, py2);
					ctx.lineTo(px3, py3);
					ctx.lineTo(px1, py1);
					ctx.closePath();
					ctx.fillStyle = `hsla(${hue},${sat}%,${lum}%,${alpha})`;
					ctx.fill();
					ctx.strokeStyle = `hsla(${hue},${sat}%,${lum + 20}%,0.15)`;
					ctx.lineWidth = 0.5;
					ctx.stroke();
				}
			}
		}

		function drawOptionsChain() {
			const chainX = W * 0.04;
			const chainY = H * 0.12;
			const cellW = (W * 0.28) / EXPIRIES.length;
			const cellH = 20;
			const strike6 = 42000; // ATM

			// Header row
			ctx.fillStyle = 'rgba(4,12,28,0.92)';
			ctx.fillRect(chainX - 4, chainY - 18, cellW * EXPIRIES.length + STRIKES.length * 0 + 80, STRIKES.length * cellH + 24);

			ctx.font = 'bold 9px var(--font-mono)';
			ctx.fillStyle = 'rgba(100,160,255,0.6)';
			ctx.textAlign = 'left';
			ctx.fillText('STRIKE', chainX, chainY - 4);

			EXPIRIES.forEach((exp, ei) => {
				ctx.textAlign = 'center';
				ctx.fillText(exp, chainX + 70 + ei * cellW + cellW / 2, chainY - 4);
			});

			STRIKES.forEach((strike, si) => {
				const isATM = strike === 42000;
				const rowY = chainY + si * cellH;

				// Strike label
				ctx.fillStyle = isATM ? 'rgba(255,200,60,0.9)' : 'rgba(160,190,240,0.55)';
				ctx.font = `${isATM ? 'bold ' : ''}9px var(--font-mono)`;
				ctx.textAlign = 'left';
				ctx.fillText(strike.toLocaleString(), chainX, rowY + 13);

				if (isATM) {
					ctx.fillStyle = 'rgba(255,200,60,0.06)';
					ctx.fillRect(chainX - 4, rowY + 2, cellW * EXPIRIES.length + 74, cellH - 2);
				}

				EXPIRIES.forEach((exp, ei) => {
					const cell = chain.find(c2 => c2.strike === strike && c2.expiry === exp);
					if (!cell) return;

					const bx = chainX + 70 + ei * cellW;
					const ivPct = (cell.iv * 100).toFixed(0);
					const ivHue = 200 - Math.min(1, cell.iv / 1.2) * 200;
					const ivAlpha = 0.4 + Math.min(1, cell.iv / 1.2) * 0.5;

					// IV cell bg
					ctx.fillStyle = `hsla(${ivHue},70%,40%,${ivAlpha * 0.3})`;
					ctx.fillRect(bx, rowY + 2, cellW - 2, cellH - 4);

					ctx.fillStyle = `hsla(${ivHue},80%,70%,${ivAlpha * 0.9 + Math.sin(time * 3 + si + ei) * 0.05})`;
					ctx.font = '9px var(--font-mono)';
					ctx.textAlign = 'center';
					ctx.fillText(ivPct + '%', bx + cellW / 2, rowY + 13);
				});
			});
		}

		function drawStats() {
			const sx = W * 0.04;
			const sy = H * 0.74;

			const stats = [
				{ label: 'VIX', val: vixVal.toFixed(2), color: vixVal > 30 ? '#ff3c50' : vixVal > 20 ? '#ffcc00' : '#00d264' },
				{ label: 'SKEW', val: '112.4', color: 'rgba(160,200,255,0.8)' },
				{ label: 'PUT/CALL', val: '0.84', color: 'rgba(160,200,255,0.8)' },
				{ label: 'IV RANK', val: '68%', color: '#ffcc00' },
				{ label: 'HV30', val: '22.1%', color: 'rgba(160,200,255,0.8)' },
			];

			stats.forEach((s, i) => {
				const bx = sx + i * 120;
				ctx.fillStyle = 'rgba(4,12,28,0.85)';
				ctx.fillRect(bx, sy, 110, 44);
				ctx.strokeStyle = 'rgba(40,80,140,0.3)';
				ctx.lineWidth = 1;
				ctx.strokeRect(bx, sy, 110, 44);

				ctx.font = '8px var(--font-mono)';
				ctx.fillStyle = 'rgba(100,140,200,0.5)';
				ctx.textAlign = 'left';
				ctx.fillText(s.label, bx + 8, sy + 14);

				ctx.font = 'bold 16px var(--font-mono)';
				ctx.fillStyle = s.color;
				ctx.fillText(s.val, bx + 8, sy + 34);
			});
		}

		function draw() {
			ctx.clearRect(0, 0, W, H);
			time += 0.016;

			ctx.fillStyle = '#030810';
			ctx.fillRect(0, 0, W, H);

			drawSurface();
			drawOptionsChain();
			drawStats();

			raf = requestAnimationFrame(draw);
		}
		draw();

		// Entrance
		gsap.set([titleRef, subtitleRef, ctaRef, statsRef], { autoAlpha: 0, x: -30 });
		const tl = gsap.timeline({ delay: 0.3 });
		tl.to(titleRef, { autoAlpha: 1, x: 0, duration: 0.8, ease: 'expo.out' })
			.to(subtitleRef, { autoAlpha: 1, x: 0, duration: 0.7, ease: 'expo.out' }, '-=0.4')
			.to(statsRef, { autoAlpha: 1, x: 0, duration: 0.7, ease: 'expo.out' }, '-=0.3')
			.to(ctaRef, { autoAlpha: 1, x: 0, duration: 0.6, ease: 'expo.out' }, '-=0.3');

		const onResize = () => {
			W = canvas.width = window.innerWidth;
			H = canvas.height = window.innerHeight;
		};
		window.addEventListener('resize', onResize);

		return () => {
			cancelAnimationFrame(raf);
			clearInterval(vixInterval);
			window.removeEventListener('resize', onResize);
		};
	});
</script>

<section class="ht4" bind:this={section}>
	<canvas bind:this={canvas}></canvas>

	<div class="ht4__ui">
		<div class="ht4__title-block" bind:this={titleRef}>
			<div class="ht4__eyebrow">IMPLIED VOLATILITY SURFACE</div>
			<h2 class="ht4__title">OPTIONS<br/>INTELLIGENCE</h2>
		</div>

		<p class="ht4__subtitle" bind:this={subtitleRef}>
			Real-time IV surface. Smile detected.<br />
			Skew signals institutional hedging.
		</p>

		<div class="ht4__stats" bind:this={statsRef}>
			<div class="ht4__stat-pill ht4__warn">⚠ ELEVATED VOL REGIME</div>
			<div class="ht4__stat-pill">TERM STRUCTURE: BACKWARDATION</div>
		</div>

		<div class="ht4__cta" bind:this={ctaRef}>
			<button class="ht4__btn">Analyze Chain</button>
		</div>
	</div>

	<div class="ht4__corner-label">BTC OPTIONS · DERIBIT LIVE</div>
	<div class="ht4__vignette"></div>
</section>

<style>
	.ht4 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #030810;
		display: flex;
		align-items: center;
	}

	canvas {
		position: absolute;
		inset: 0;
		width: 100%; height: 100%;
		z-index: 2;
	}

	.ht4__ui {
		position: absolute;
		right: 4vw;
		top: 50%;
		transform: translateY(-50%);
		z-index: 10;
		display: flex;
		flex-direction: column;
		gap: 1.2rem;
		max-width: 320px;
	}

	.ht4__eyebrow {
		font-family: var(--font-mono);
		font-size: 0.6rem;
		letter-spacing: 0.2em;
		color: rgba(100,160,255,0.55);
		border-left: 2px solid rgba(100,160,255,0.3);
		padding-left: 0.7rem;
	}

	.ht4__title {
		font-family: var(--font-cinematic);
		font-size: clamp(2.5rem, 5vw, 4.5rem);
		line-height: 0.9;
		letter-spacing: 0.06em;
		color: #fff;
		text-shadow: 0 0 40px rgba(80,140,255,0.3);
	}

	.ht4__subtitle {
		font-size: 0.82rem;
		color: rgba(160,190,240,0.55);
		line-height: 1.7;
		font-weight: 300;
	}

	.ht4__stats {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	.ht4__stat-pill {
		font-family: var(--font-mono);
		font-size: 0.62rem;
		letter-spacing: 0.1em;
		color: rgba(140,180,255,0.65);
		border: 1px solid rgba(80,140,255,0.2);
		padding: 0.3rem 0.7rem;
		width: fit-content;
	}

	.ht4__warn {
		color: rgba(255,200,60,0.8) !important;
		border-color: rgba(255,200,60,0.25) !important;
	}

	.ht4__cta { margin-top: 0.5rem; }

	.ht4__btn {
		padding: 0.7rem 1.8rem;
		background: rgba(60,120,255,0.15);
		border: 1px solid rgba(80,150,255,0.4);
		color: rgba(160,200,255,0.9);
		font-family: var(--font-mono);
		font-size: 0.75rem;
		letter-spacing: 0.12em;
		cursor: pointer;
		transition: all 0.2s;
	}

	.ht4__btn:hover {
		background: rgba(60,120,255,0.3);
		box-shadow: 0 0 24px rgba(80,150,255,0.3);
	}

	.ht4__corner-label {
		position: absolute;
		bottom: 1.5rem;
		left: 50%;
		transform: translateX(-50%);
		font-family: var(--font-mono);
		font-size: 0.58rem;
		letter-spacing: 0.2em;
		color: rgba(80,120,200,0.3);
		z-index: 10;
	}

	.ht4__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at 70% 50%, transparent 35%, rgba(3,8,16,0.75) 100%);
		pointer-events: none;
		z-index: 5;
	}
</style>
