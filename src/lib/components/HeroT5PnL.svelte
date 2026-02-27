<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { RocketLaunch, TrendUp, ArrowRight } from 'phosphor-svelte';

	let section: HTMLElement;
	let canvas: HTMLCanvasElement;
	let titleRef: HTMLElement;
	let pnlRef: HTMLElement;
	let pnlNumRef: HTMLElement;
	let ctaRef: HTMLElement;
	let statsRef: HTMLElement;
	let tradeListRef: HTMLElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let W = (canvas.width = window.innerWidth);
		let H = (canvas.height = window.innerHeight);
		let raf: number;
		let time = 0;

		// Portfolio equity curve
		const POINTS = 200;
		let equityCurve: number[] = [];
		let val = 10000;
		for (let i = 0; i < POINTS; i++) {
			const drift = 0.0015 + (Math.random() - 0.42) * 0.025;
			val *= (1 + drift);
			equityCurve.push(val);
		}

		// Rocket particles
		type Rocket = { x: number; y: number; vy: number; vx: number; alpha: number; size: number; trail: [number,number][] };
		const rockets: Rocket[] = [];
		function spawnRocket() {
			rockets.push({
				x: W * 0.3 + Math.random() * W * 0.4,
				y: H + 20,
				vx: (Math.random() - 0.5) * 1.5,
				vy: -(6 + Math.random() * 8),
				alpha: 1,
				size: 3 + Math.random() * 4,
				trail: [],
			});
		}

		// Gold coin burst particles
		type Coin = { x: number; y: number; vx: number; vy: number; alpha: number; size: number; rot: number; vrot: number };
		const coins: Coin[] = [];
		function spawnCoins(x: number, y: number) {
			for (let i = 0; i < 18; i++) {
				const angle = Math.random() * Math.PI * 2;
				const speed = 2 + Math.random() * 8;
				coins.push({
					x, y,
					vx: Math.cos(angle) * speed,
					vy: Math.sin(angle) * speed - 4,
					alpha: 1,
					size: 4 + Math.random() * 6,
					rot: Math.random() * Math.PI * 2,
					vrot: (Math.random() - 0.5) * 0.4,
				});
			}
		}

		// P&L counter animation
		let displayPnL = 0;
		let targetPnL = 0;
		let pnlVelocity = 0;

		// Live trades feed
		const tradeSymbols = ['NVDA', 'TSLA', 'SPY', 'QQQ', 'AAPL', 'META', 'MSFT', 'AMD', 'COIN', 'AMZN'];
		const trades: { symbol: string; pnl: number; time: string; dir: string }[] = [];
		const tradeInterval = setInterval(() => {
			const sym = tradeSymbols[Math.floor(Math.random() * tradeSymbols.length)];
			const pnl = (Math.random() - 0.35) * 4200;
			const now = new Date();
			trades.unshift({
				symbol: sym,
				pnl,
				time: `${now.getHours()}:${String(now.getMinutes()).padStart(2,'0')}:${String(now.getSeconds()).padStart(2,'0')}`,
				dir: Math.random() > 0.4 ? 'LONG' : 'SHORT',
			});
			if (trades.length > 6) trades.pop();
			targetPnL += pnl;

			if (pnl > 800) spawnCoins(W * 0.5, H * 0.55);
		}, 1200);

		// Rockets spawn periodically
		const rocketInterval = setInterval(spawnRocket, 1800);
		spawnRocket();

		function drawEquityCurve() {
			if (equityCurve.length < 2) return;
			const chartX = W * 0.35;
			const chartW = W * 0.6;
			const chartY = H * 0.15;
			const chartH = H * 0.55;

			const minV = Math.min(...equityCurve);
			const maxV = Math.max(...equityCurve);
			const range = maxV - minV || 1;

			// Live extension
			const lastVal = equityCurve[equityCurve.length - 1];
			const newVal = lastVal * (1 + (Math.random() - 0.42) * 0.006);
			equityCurve.push(newVal);
			if (equityCurve.length > POINTS + 100) equityCurve.shift();

			// Area fill
			ctx.beginPath();
			equityCurve.forEach((v, i) => {
				const x = chartX + (i / equityCurve.length) * chartW;
				const y = chartY + chartH - ((v - minV) / range) * chartH;
				i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y);
			});
			ctx.lineTo(chartX + chartW, chartY + chartH);
			ctx.lineTo(chartX, chartY + chartH);
			ctx.closePath();
			const areaGrad = ctx.createLinearGradient(0, chartY, 0, chartY + chartH);
			areaGrad.addColorStop(0, 'rgba(0,210,100,0.22)');
			areaGrad.addColorStop(0.6, 'rgba(0,210,100,0.06)');
			areaGrad.addColorStop(1, 'rgba(0,210,100,0)');
			ctx.fillStyle = areaGrad;
			ctx.fill();

			// Line
			ctx.beginPath();
			equityCurve.forEach((v, i) => {
				const x = chartX + (i / equityCurve.length) * chartW;
				const y = chartY + chartH - ((v - minV) / range) * chartH;
				i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y);
			});
			ctx.strokeStyle = 'rgba(0,210,100,0.85)';
			ctx.lineWidth = 2;
			ctx.stroke();

			// Glow dot at end
			const lastX = chartX + chartW;
			const lastY = chartY + chartH - ((newVal - minV) / range) * chartH;
			const pulse = 0.6 + Math.sin(time * 6) * 0.4;
			ctx.beginPath();
			ctx.arc(lastX, lastY, 5, 0, Math.PI * 2);
			ctx.fillStyle = '#00d264';
			ctx.shadowColor = '#00d264';
			ctx.shadowBlur = 14 * pulse;
			ctx.fill();
			ctx.shadowBlur = 0;

			// Grid lines
			for (let g = 0; g <= 4; g++) {
				const gy = chartY + (g / 4) * chartH;
				ctx.beginPath();
				ctx.moveTo(chartX, gy);
				ctx.lineTo(chartX + chartW, gy);
				ctx.strokeStyle = 'rgba(40,80,140,0.15)';
				ctx.lineWidth = 1;
				ctx.stroke();
				const gVal = maxV - (g / 4) * range;
				ctx.fillStyle = 'rgba(100,140,200,0.35)';
				ctx.font = '9px var(--font-mono)';
				ctx.textAlign = 'right';
				ctx.fillText('$' + gVal.toFixed(0), chartX - 6, gy + 4);
			}

			// Return label
			const totalReturn = ((newVal / equityCurve[0] - 1) * 100).toFixed(1);
			ctx.font = 'bold 11px var(--font-mono)';
			ctx.fillStyle = 'rgba(0,210,100,0.8)';
			ctx.textAlign = 'left';
			ctx.fillText(`+${totalReturn}% ALL TIME`, chartX, chartY - 8);
		}

		function drawRockets() {
			for (let i = rockets.length - 1; i >= 0; i--) {
				const r = rockets[i];
				r.trail.push([r.x, r.y]);
				if (r.trail.length > 20) r.trail.shift();
				r.x += r.vx;
				r.y += r.vy;
				r.vy *= 0.995;
				r.alpha -= 0.004;
				if (r.y < -50 || r.alpha <= 0) { rockets.splice(i, 1); continue; }

				// Trail
				for (let t = 0; t < r.trail.length; t++) {
					const [tx, ty] = r.trail[t];
					const ta = (t / r.trail.length) * r.alpha * 0.5;
					ctx.beginPath();
					ctx.arc(tx, ty, r.size * (t / r.trail.length) * 0.6, 0, Math.PI * 2);
					ctx.fillStyle = `rgba(255,${150 + t * 4},0,${ta})`;
					ctx.shadowColor = 'rgba(255,180,0,0.5)';
					ctx.shadowBlur = 6;
					ctx.fill();
				}
				ctx.shadowBlur = 0;

				// Rocket body
				ctx.save();
				ctx.translate(r.x, r.y);
				ctx.rotate(Math.atan2(r.vy, r.vx) + Math.PI / 2);
				ctx.beginPath();
				ctx.moveTo(0, -r.size * 2);
				ctx.lineTo(r.size * 0.8, r.size);
				ctx.lineTo(-r.size * 0.8, r.size);
				ctx.closePath();
				ctx.fillStyle = `rgba(255,220,80,${r.alpha * 0.9})`;
				ctx.fill();
				ctx.restore();
			}
		}

		function drawCoins() {
			for (let i = coins.length - 1; i >= 0; i--) {
				const c = coins[i];
				c.x += c.vx;
				c.y += c.vy;
				c.vy += 0.18;
				c.vx *= 0.98;
				c.rot += c.vrot;
				c.alpha -= 0.016;
				if (c.alpha <= 0) { coins.splice(i, 1); continue; }

				ctx.save();
				ctx.translate(c.x, c.y);
				ctx.rotate(c.rot);
				ctx.globalAlpha = c.alpha;
				ctx.beginPath();
				ctx.ellipse(0, 0, c.size, c.size * Math.abs(Math.cos(c.rot * 3)) + 1, 0, 0, Math.PI * 2);
				ctx.fillStyle = '#ffd700';
				ctx.shadowColor = '#ffd700';
				ctx.shadowBlur = 8;
				ctx.fill();
				ctx.strokeStyle = '#ff9900';
				ctx.lineWidth = 1;
				ctx.stroke();
				ctx.restore();
			}
			ctx.shadowBlur = 0;
			ctx.globalAlpha = 1;
		}

		function drawTrades() {
			const tx = W * 0.04;
			let ty = H * 0.55;

			ctx.fillStyle = 'rgba(3,10,22,0.0)';
			ctx.font = 'bold 9px var(--font-mono)';
			ctx.fillStyle = 'rgba(80,120,200,0.4)';
			ctx.textAlign = 'left';
			ctx.fillText('RECENT FILLS', tx, ty - 8);

			trades.forEach((tr, i) => {
				const bull = tr.pnl >= 0;
				const rowY = ty + i * 28;
				ctx.fillStyle = 'rgba(4,12,28,0.8)';
				ctx.fillRect(tx - 4, rowY, 200, 24);
				ctx.strokeStyle = bull ? 'rgba(0,210,100,0.2)' : 'rgba(255,60,80,0.2)';
				ctx.lineWidth = 1;
				ctx.strokeRect(tx - 4, rowY, 200, 24);

				ctx.fillStyle = 'rgba(200,220,255,0.8)';
				ctx.font = 'bold 9px var(--font-mono)';
				ctx.fillText(tr.symbol, tx, rowY + 15);
				ctx.fillStyle = 'rgba(100,140,200,0.4)';
				ctx.font = '8px var(--font-mono)';
				ctx.fillText(tr.dir, tx + 42, rowY + 15);
				ctx.fillStyle = bull ? 'rgba(0,210,100,0.85)' : 'rgba(255,60,80,0.85)';
				ctx.font = 'bold 9px var(--font-mono)';
				ctx.textAlign = 'right';
				ctx.fillText((bull ? '+' : '') + '$' + tr.pnl.toFixed(0), tx + 196, rowY + 15);
				ctx.textAlign = 'left';
				ctx.fillStyle = 'rgba(80,110,160,0.35)';
				ctx.font = '7px var(--font-mono)';
				ctx.fillText(tr.time, tx + 42, rowY + 15 - 6);
			});
		}

		// P&L smooth counter
		function updatePnL() {
			pnlVelocity += (targetPnL - displayPnL) * 0.04;
			pnlVelocity *= 0.88;
			displayPnL += pnlVelocity;
			if (pnlNumRef) {
				const val2 = displayPnL;
				pnlNumRef.textContent = (val2 >= 0 ? '+$' : '-$') + Math.abs(val2).toFixed(2);
				pnlNumRef.style.color = val2 >= 0 ? '#00d264' : '#ff3c50';
				pnlNumRef.style.textShadow = val2 >= 0
					? '0 0 40px rgba(0,210,100,0.6), 0 0 80px rgba(0,210,100,0.2)'
					: '0 0 40px rgba(255,60,80,0.6)';
			}
		}

		function draw() {
			ctx.clearRect(0, 0, W, H);
			time += 0.016;

			const bg = ctx.createLinearGradient(0, 0, 0, H);
			bg.addColorStop(0, '#030d10');
			bg.addColorStop(1, '#020810');
			ctx.fillStyle = bg;
			ctx.fillRect(0, 0, W, H);

			// Subtle grid
			for (let gx = 0; gx < W; gx += 80) {
				ctx.beginPath();
				ctx.moveTo(gx, 0); ctx.lineTo(gx, H);
				ctx.strokeStyle = 'rgba(0,100,50,0.05)';
				ctx.lineWidth = 1;
				ctx.stroke();
			}

			drawEquityCurve();
			drawRockets();
			drawCoins();
			drawTrades();
			updatePnL();

			raf = requestAnimationFrame(draw);
		}
		draw();

		// Entrance
		gsap.set([titleRef, pnlRef, statsRef, ctaRef], { autoAlpha: 0, y: 30 });
		const tl = gsap.timeline({ delay: 0.3 });
		tl.to(titleRef, { autoAlpha: 1, y: 0, duration: 0.8, ease: 'expo.out' })
			.to(pnlRef, { autoAlpha: 1, y: 0, duration: 0.9, ease: 'expo.out' }, '-=0.4')
			.to(statsRef, { autoAlpha: 1, y: 0, duration: 0.7, ease: 'expo.out' }, '-=0.4')
			.to(ctaRef, { autoAlpha: 1, y: 0, duration: 0.6, ease: 'expo.out' }, '-=0.3');

		// Blast-off: P&L counter launch
		gsap.delayedCall(0.8, () => {
			let seed = 0;
			const blastInterval = setInterval(() => {
				seed += (Math.random() > 0.45 ? 1 : -1) * (800 + Math.random() * 3200);
				targetPnL = seed;
				if (Math.abs(seed) > 20000) {
					clearInterval(blastInterval);
					targetPnL = 48392.77;
				}
			}, 120);
		});

		const onResize = () => {
			W = canvas.width = window.innerWidth;
			H = canvas.height = window.innerHeight;
		};
		window.addEventListener('resize', onResize);

		return () => {
			cancelAnimationFrame(raf);
			clearInterval(tradeInterval);
			clearInterval(rocketInterval);
			window.removeEventListener('resize', onResize);
		};
	});
</script>

<section class="ht5" bind:this={section}>
	<canvas bind:this={canvas}></canvas>

	<div class="ht5__left">
		<div class="ht5__eyebrow" bind:this={titleRef}>
			<RocketLaunch size={14} weight="fill" />
			<span>PORTFOLIO · LIVE P&L</span>
		</div>

		<div class="ht5__pnl-block" bind:this={pnlRef}>
			<div class="ht5__pnl-label">TODAY'S PROFIT</div>
			<div class="ht5__pnl-num" bind:this={pnlNumRef}>+$0.00</div>
		</div>

		<div class="ht5__stats" bind:this={statsRef}>
			<div class="ht5__stat">
				<span class="ht5__stat-l">WIN RATE</span>
				<span class="ht5__stat-v ht5__bull">74.2%</span>
			</div>
			<div class="ht5__stat">
				<span class="ht5__stat-l">AVG WIN</span>
				<span class="ht5__stat-v ht5__bull">$1,842</span>
			</div>
			<div class="ht5__stat">
				<span class="ht5__stat-l">SHARPE</span>
				<span class="ht5__stat-v">3.81</span>
			</div>
			<div class="ht5__stat">
				<span class="ht5__stat-l">MAX DD</span>
				<span class="ht5__stat-v ht5__bear">-4.2%</span>
			</div>
		</div>

		<div class="ht5__cta" bind:this={ctaRef}>
			<button class="ht5__btn">
				<TrendUp size={15} weight="bold" />
				<span>Copy Trades</span>
				<ArrowRight size={14} weight="bold" />
			</button>
		</div>
	</div>

	<div class="ht5__vignette"></div>
</section>

<style>
	.ht5 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #020810;
		display: flex;
		align-items: center;
	}

	canvas {
		position: absolute;
		inset: 0;
		width: 100%; height: 100%;
		z-index: 2;
		pointer-events: none;
	}

	.ht5__left {
		position: relative;
		z-index: 10;
		padding: 0 0 0 4vw;
		display: flex;
		flex-direction: column;
		gap: 1.4rem;
	}

	.ht5__eyebrow {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		font-family: var(--font-mono);
		font-size: 0.65rem;
		letter-spacing: 0.18em;
		color: rgba(0, 210, 100, 0.65);
		border-bottom: 1px solid rgba(0, 210, 100, 0.2);
		padding-bottom: 0.4rem;
		width: fit-content;
	}

	.ht5__pnl-block {
		display: flex;
		flex-direction: column;
		gap: 0.3rem;
	}

	.ht5__pnl-label {
		font-family: var(--font-mono);
		font-size: 0.62rem;
		letter-spacing: 0.18em;
		color: rgba(100, 160, 100, 0.45);
	}

	.ht5__pnl-num {
		font-family: var(--font-mono);
		font-size: clamp(2.5rem, 5vw, 4.5rem);
		font-weight: 700;
		color: #00d264;
		letter-spacing: 0.04em;
		line-height: 1;
		text-shadow: 0 0 40px rgba(0, 210, 100, 0.5);
		transition: color 0.2s;
	}

	.ht5__stats {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 0.6rem 1.6rem;
	}

	.ht5__stat {
		display: flex;
		flex-direction: column;
		gap: 2px;
	}

	.ht5__stat-l {
		font-family: var(--font-mono);
		font-size: 0.55rem;
		letter-spacing: 0.12em;
		color: rgba(80, 120, 180, 0.45);
	}

	.ht5__stat-v {
		font-family: var(--font-mono);
		font-size: 0.85rem;
		font-weight: 700;
		color: rgba(180, 210, 255, 0.85);
		letter-spacing: 0.06em;
	}

	.ht5__bull { color: #00d264 !important; }
	.ht5__bear { color: #ff3c50 !important; }

	.ht5__cta { margin-top: 0.4rem; }

	.ht5__btn {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0.75rem 1.8rem;
		background: rgba(0, 210, 100, 0.12);
		border: 1px solid rgba(0, 210, 100, 0.4);
		color: rgba(100, 230, 150, 0.95);
		font-family: var(--font-mono);
		font-size: 0.78rem;
		font-weight: 600;
		letter-spacing: 0.08em;
		cursor: pointer;
		transition: all 0.22s;
		box-shadow: 0 0 20px rgba(0, 210, 100, 0.1);
	}

	.ht5__btn:hover {
		background: rgba(0, 210, 100, 0.24);
		box-shadow: 0 0 40px rgba(0, 210, 100, 0.3);
		transform: translateY(-2px);
	}

	.ht5__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at 25% 50%, transparent 30%, rgba(2,8,16,0.82) 100%);
		pointer-events: none;
		z-index: 6;
	}
</style>
