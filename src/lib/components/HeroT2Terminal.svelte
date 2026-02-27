<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';

	let section: HTMLElement;
	let canvas: HTMLCanvasElement;
	let titleRef: HTMLElement;
	let ctaRef: HTMLElement;

	type MiniCandle = { o: number; c: number; h: number; l: number };
	type Chart = {
		symbol: string; price: number; change: number;
		candles: MiniCandle[]; x: number; y: number; w: number; h: number;
		hue: number;
	};

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let W = (canvas.width = window.innerWidth);
		let H = (canvas.height = window.innerHeight);
		let raf: number;
		let time = 0;

		// Generate candles for a symbol
		function genCandles(base: number, count: number, bullish: boolean): MiniCandle[] {
			const arr: MiniCandle[] = [];
			let price = base;
			for (let i = 0; i < count; i++) {
				const bias = bullish ? 0.55 : 0.45;
				const drift = (Math.random() - (1 - bias)) * base * 0.025;
				const o = price;
				const c = o + drift;
				const h = Math.max(o, c) + Math.random() * base * 0.012;
				const l = Math.min(o, c) - Math.random() * base * 0.012;
				arr.push({ o, c, h, l });
				price = c;
			}
			return arr;
		}

		const CHARTS: Chart[] = [
			{ symbol: 'SPY', price: 524.18, change: 1.24, candles: genCandles(524, 40, true), x: 0, y: 0, w: 0, h: 0, hue: 140 },
			{ symbol: 'QQQ', price: 438.72, change: 2.01, candles: genCandles(438, 40, true), x: 0, y: 0, w: 0, h: 0, hue: 200 },
			{ symbol: 'TSLA', price: 248.45, change: -3.18, candles: genCandles(248, 40, false), x: 0, y: 0, w: 0, h: 0, hue: 0 },
			{ symbol: 'NVDA', price: 876.32, change: 4.55, candles: genCandles(876, 40, true), x: 0, y: 0, w: 0, h: 0, hue: 120 },
			{ symbol: 'AAPL', price: 189.64, change: 0.87, candles: genCandles(189, 40, true), x: 0, y: 0, w: 0, h: 0, hue: 180 },
			{ symbol: 'BTC', price: 42380, change: -1.44, candles: genCandles(42380, 40, false), x: 0, y: 0, w: 0, h: 0, hue: 30 },
		];

		// Layout: 3x2 grid
		function layoutCharts() {
			const cols = 3, rows = 2;
			const padX = W * 0.02, padY = H * 0.15;
			const cellW = (W - padX * (cols + 1)) / cols;
			const cellH = (H - padY - padX * (rows + 1)) / rows;
			CHARTS.forEach((ch, i) => {
				const col = i % cols;
				const row = Math.floor(i / cols);
				ch.x = padX + col * (cellW + padX);
				ch.y = padY + row * (cellH + padX);
				ch.w = cellW;
				ch.h = cellH;
			});
		}
		layoutCharts();

		// Tick simulation
		CHARTS.forEach(ch => {
			setInterval(() => {
				const move = (Math.random() - (ch.change < 0 ? 0.52 : 0.48)) * ch.price * 0.004;
				ch.price += move;
				ch.change += move / ch.price * 100 * (Math.random() * 0.3);
				const last = ch.candles[ch.candles.length - 1];
				const newC = last.c + move;
				ch.candles.push({
					o: last.c, c: newC,
					h: Math.max(last.c, newC) + Math.random() * ch.price * 0.003,
					l: Math.min(last.c, newC) - Math.random() * ch.price * 0.003,
				});
				if (ch.candles.length > 50) ch.candles.shift();
			}, 300 + Math.random() * 400);
		});

		function drawMiniChart(ch: Chart) {
			const { x, y, w, h, candles, symbol, price, change, hue } = ch;
			const bull = change >= 0;
			const color = bull ? '#00d264' : '#ff3c50';
			const colorA = bull ? 'rgba(0,210,100,' : 'rgba(255,60,80,';

			// Panel bg
			const bg = ctx.createLinearGradient(x, y, x, y + h);
			bg.addColorStop(0, `hsla(${hue},30%,6%,0.95)`);
			bg.addColorStop(1, 'rgba(4,10,20,0.97)');
			ctx.fillStyle = bg;
			ctx.beginPath();
			ctx.roundRect(x, y, w, h, 6);
			ctx.fill();

			// Border glow
			ctx.strokeStyle = bull ? 'rgba(0,210,100,0.2)' : 'rgba(255,60,80,0.2)';
			ctx.lineWidth = 1;
			ctx.stroke();

			// Header
			ctx.fillStyle = 'rgba(200,220,255,0.9)';
			ctx.font = `bold 13px var(--font-mono)`;
			ctx.textAlign = 'left';
			ctx.fillText(symbol, x + 12, y + 22);

			ctx.fillStyle = color;
			ctx.font = `bold 12px var(--font-mono)`;
			ctx.textAlign = 'right';
			ctx.fillText('$' + price.toFixed(price > 1000 ? 0 : 2), x + w - 12, y + 22);

			const pct = (bull ? '+' : '') + change.toFixed(2) + '%';
			ctx.font = `10px var(--font-mono)`;
			ctx.fillText(pct, x + w - 12, y + 36);

			// Candle area
			const padL = 8, padR = 8, padT = 48, padB = 28;
			const cAreaW = w - padL - padR;
			const cAreaH = h - padT - padB;
			const cAreaX = x + padL;
			const cAreaY = y + padT;

			const hi = Math.max(...candles.map(c => c.h));
			const lo = Math.min(...candles.map(c => c.l));
			const range = hi - lo || 1;

			const cw = Math.max(2, cAreaW / candles.length - 1);

			// Area fill under close line
			ctx.beginPath();
			candles.forEach((c, i) => {
				const cx2 = cAreaX + (i / candles.length) * cAreaW;
				const cy2 = cAreaY + cAreaH - ((c.c - lo) / range) * cAreaH;
				i === 0 ? ctx.moveTo(cx2, cy2) : ctx.lineTo(cx2, cy2);
			});
			ctx.lineTo(cAreaX + cAreaW, cAreaY + cAreaH);
			ctx.lineTo(cAreaX, cAreaY + cAreaH);
			ctx.closePath();
			const areaGrad = ctx.createLinearGradient(cAreaX, cAreaY, cAreaX, cAreaY + cAreaH);
			areaGrad.addColorStop(0, colorA + '0.12)');
			areaGrad.addColorStop(1, colorA + '0)');
			ctx.fillStyle = areaGrad;
			ctx.fill();

			// Candles
			for (let i = 0; i < candles.length; i++) {
				const c = candles[i];
				const cx2 = cAreaX + (i / candles.length) * cAreaW;
				const oY = cAreaY + cAreaH - ((c.o - lo) / range) * cAreaH;
				const cY = cAreaY + cAreaH - ((c.c - lo) / range) * cAreaH;
				const hY = cAreaY + cAreaH - ((c.h - lo) / range) * cAreaH;
				const lY = cAreaY + cAreaH - ((c.l - lo) / range) * cAreaH;
				const cb = c.c >= c.o;

				ctx.strokeStyle = cb ? 'rgba(0,210,100,0.5)' : 'rgba(255,60,80,0.5)';
				ctx.lineWidth = 0.8;
				ctx.beginPath();
				ctx.moveTo(cx2 + cw / 2, hY);
				ctx.lineTo(cx2 + cw / 2, lY);
				ctx.stroke();

				ctx.fillStyle = cb ? 'rgba(0,190,85,0.8)' : 'rgba(240,50,70,0.8)';
				ctx.fillRect(cx2, Math.min(oY, cY), Math.max(1, cw - 0.5), Math.max(1, Math.abs(cY - oY)));
			}

			// Sparkle on last candle if live
			const isFlashing = Math.sin(time * 8 + hue) > 0.8;
			if (isFlashing) {
				const lastC = candles[candles.length - 1];
				const lx = cAreaX + ((candles.length - 1) / candles.length) * cAreaW + cw / 2;
				const ly = cAreaY + cAreaH - ((lastC.c - lo) / range) * cAreaH;
				ctx.beginPath();
				ctx.arc(lx, ly, 3, 0, Math.PI * 2);
				ctx.fillStyle = color;
				ctx.shadowColor = color;
				ctx.shadowBlur = 10;
				ctx.fill();
				ctx.shadowBlur = 0;
			}

			// Bottom separator
			ctx.fillStyle = 'rgba(40,80,140,0.2)';
			ctx.fillRect(x, y + h - padB, w, 1);
		}

		// Scrolling ticker tape
		const tickerItems = [
			'SPY +1.24%', 'QQQ +2.01%', 'TSLA -3.18%', 'NVDA +4.55%', 'AAPL +0.87%',
			'BTC -1.44%', 'ETH +2.33%', 'AMZN +1.76%', 'MSFT +0.94%', 'META -0.52%',
			'GOOGL +1.12%', 'NFLX +3.88%', 'AMD +5.21%', 'COIN -2.17%', 'PLTR +6.44%',
		];
		let tickerOffset = 0;

		function drawTicker() {
			const tickY = H - 32;
			ctx.fillStyle = 'rgba(4,12,28,0.95)';
			ctx.fillRect(0, tickY, W, 32);
			ctx.strokeStyle = 'rgba(40,80,140,0.4)';
			ctx.lineWidth = 1;
			ctx.beginPath();
			ctx.moveTo(0, tickY);
			ctx.lineTo(W, tickY);
			ctx.stroke();

			tickerOffset -= 1.2;
			const itemW = 130;
			const totalW = tickerItems.length * itemW;
			if (tickerOffset < -totalW) tickerOffset = 0;

			ctx.save();
			ctx.beginPath();
			ctx.rect(0, tickY, W, 32);
			ctx.clip();

			for (let rep = 0; rep < 3; rep++) {
				tickerItems.forEach((item, i) => {
					const tx = tickerOffset + rep * totalW + i * itemW;
					if (tx < -itemW || tx > W + itemW) return;
					const bull = item.includes('+');
					ctx.fillStyle = bull ? 'rgba(0,210,100,0.85)' : 'rgba(255,60,80,0.85)';
					ctx.font = '10px var(--font-mono)';
					ctx.textAlign = 'left';
					ctx.fillText(item, tx, tickY + 20);
					ctx.fillStyle = 'rgba(40,80,140,0.3)';
					ctx.fillRect(tx + itemW - 10, tickY + 8, 1, 16);
				});
			}
			ctx.restore();
		}

		function drawHeader() {
			// Top bar
			ctx.fillStyle = 'rgba(4,10,22,0.97)';
			ctx.fillRect(0, 0, W, H * 0.13);

			// Animated scanner line
			const scanX = (time * 80) % W;
			const scanGrad = ctx.createLinearGradient(scanX - 80, 0, scanX + 80, 0);
			scanGrad.addColorStop(0, 'rgba(60,140,255,0)');
			scanGrad.addColorStop(0.5, 'rgba(60,140,255,0.15)');
			scanGrad.addColorStop(1, 'rgba(60,140,255,0)');
			ctx.fillStyle = scanGrad;
			ctx.fillRect(0, 0, W, H * 0.13);

			ctx.fillStyle = 'rgba(40,80,140,0.4)';
			ctx.fillRect(0, H * 0.13, W, 1);
		}

		function draw() {
			ctx.clearRect(0, 0, W, H);
			time += 0.016;

			ctx.fillStyle = '#030b18';
			ctx.fillRect(0, 0, W, H);

			drawHeader();
			CHARTS.forEach(ch => drawMiniChart(ch));
			drawTicker();

			raf = requestAnimationFrame(draw);
		}
		draw();

		// Entrance
		gsap.set([titleRef, ctaRef], { autoAlpha: 0, y: -20 });
		gsap.to(titleRef, { autoAlpha: 1, y: 0, duration: 0.8, delay: 0.3, ease: 'power3.out' });
		gsap.to(ctaRef, { autoAlpha: 1, y: 0, duration: 0.7, delay: 0.7, ease: 'power3.out' });

		// Chart entrance stagger
		CHARTS.forEach((ch, i) => {
			const origY = ch.y;
			ch.y = origY + 40;
			gsap.to(ch, { y: origY, duration: 0.8, delay: 0.4 + i * 0.1, ease: 'expo.out' });
		});

		const onResize = () => {
			W = canvas.width = window.innerWidth;
			H = canvas.height = window.innerHeight;
			layoutCharts();
		};
		window.addEventListener('resize', onResize);

		return () => {
			cancelAnimationFrame(raf);
			window.removeEventListener('resize', onResize);
		};
	});
</script>

<section class="ht2" bind:this={section}>
	<canvas bind:this={canvas}></canvas>

	<div class="ht2__header" bind:this={titleRef}>
		<div class="ht2__logo">⬡ APEX MARKETS</div>
		<div class="ht2__status">
			<span class="ht2__dot"></span>
			<span>LIVE</span>
		</div>
		<div class="ht2__time" id="ht2-time">NYSE · 14:32:07 EST</div>
	</div>

	<div class="ht2__cta" bind:this={ctaRef}>
		<button class="ht2__btn">Open Terminal</button>
	</div>

	<div class="ht2__scanline"></div>
</section>

<style>
	.ht2 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #030b18;
	}

	canvas {
		position: absolute;
		inset: 0;
		width: 100%; height: 100%;
		z-index: 2;
	}

	.ht2__header {
		position: absolute;
		top: 0; left: 0; right: 0;
		height: calc(13vh);
		z-index: 10;
		display: flex;
		align-items: center;
		padding: 0 2rem;
		gap: 1.5rem;
		pointer-events: none;
	}

	.ht2__logo {
		font-family: var(--font-cinematic);
		font-size: 1.4rem;
		letter-spacing: 0.1em;
		color: rgba(100, 160, 255, 0.9);
		text-shadow: 0 0 20px rgba(80, 140, 255, 0.4);
	}

	.ht2__status {
		display: flex;
		align-items: center;
		gap: 0.4rem;
		font-family: var(--font-mono);
		font-size: 0.65rem;
		letter-spacing: 0.15em;
		color: rgba(0, 210, 100, 0.8);
	}

	.ht2__dot {
		width: 6px; height: 6px;
		border-radius: 50%;
		background: #00d264;
		box-shadow: 0 0 8px rgba(0, 210, 100, 0.8);
		animation: dotPulse 1s ease-in-out infinite;
	}

	@keyframes dotPulse {
		0%, 100% { opacity: 1; box-shadow: 0 0 8px rgba(0,210,100,0.8); }
		50% { opacity: 0.4; box-shadow: 0 0 2px rgba(0,210,100,0.3); }
	}

	.ht2__time {
		font-family: var(--font-mono);
		font-size: 0.65rem;
		color: rgba(100, 140, 200, 0.5);
		letter-spacing: 0.1em;
		margin-left: auto;
	}

	.ht2__cta {
		position: absolute;
		top: calc(13vh / 2);
		right: 2rem;
		transform: translateY(-50%);
		z-index: 10;
	}

	.ht2__btn {
		padding: 0.5rem 1.4rem;
		background: rgba(60, 120, 255, 0.15);
		border: 1px solid rgba(80, 150, 255, 0.4);
		color: rgba(160, 200, 255, 0.9);
		font-family: var(--font-mono);
		font-size: 0.72rem;
		letter-spacing: 0.1em;
		cursor: pointer;
		transition: all 0.2s;
	}

	.ht2__btn:hover {
		background: rgba(60, 120, 255, 0.3);
		box-shadow: 0 0 20px rgba(80, 150, 255, 0.3);
	}

	.ht2__scanline {
		position: absolute;
		inset: 0;
		background: repeating-linear-gradient(
			0deg,
			transparent, transparent 3px,
			rgba(0, 0, 0, 0.04) 3px, rgba(0, 0, 0, 0.04) 4px
		);
		pointer-events: none;
		z-index: 15;
	}
</style>
