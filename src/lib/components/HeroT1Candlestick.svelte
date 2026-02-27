<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';

	let section: HTMLElement;
	let chartCanvas: HTMLCanvasElement;
	let depthCanvas: HTMLCanvasElement;
	let titleRef: HTMLElement;
	let statsRef: HTMLElement;
	let ctaRef: HTMLElement;
	let priceTagRef: HTMLElement;
	let tickerRef: HTMLElement;

	type Candle = {
		open: number; close: number; high: number; low: number;
		volume: number; timestamp: number;
	};

	onMount(() => {
		const cCtx = chartCanvas.getContext('2d')!;
		const dCtx = depthCanvas.getContext('2d')!;
		let W = (chartCanvas.width = depthCanvas.width = window.innerWidth);
		let H = (chartCanvas.height = window.innerHeight);
		depthCanvas.height = Math.floor(H * 0.28);
		let raf: number;
		let time = 0;

		// Generate realistic price series via Brownian motion
		const CANDLE_COUNT = 80;
		let basePrice = 42380;
		const candles: Candle[] = [];

		for (let i = 0; i < CANDLE_COUNT; i++) {
			const drift = (Math.random() - 0.48) * 180;
			const open = basePrice;
			const close = open + drift + (Math.random() - 0.5) * 60;
			const high = Math.max(open, close) + Math.random() * 120;
			const low = Math.min(open, close) - Math.random() * 120;
			const volume = 0.4 + Math.random() * 1.6;
			candles.push({ open, close, high, low, volume, timestamp: i });
			basePrice = close;
		}

		// Live candle being built tick by tick
		let liveCandle: Candle = {
			open: candles[candles.length - 1].close,
			close: candles[candles.length - 1].close,
			high: candles[candles.length - 1].close + 20,
			low: candles[candles.length - 1].close - 10,
			volume: 0.1, timestamp: CANDLE_COUNT,
		};
		let livePrice = liveCandle.close;

		// Order book depth data
		type Level = { price: number; size: number };
		const generateBook = (mid: number) => {
			const bids: Level[] = [];
			const asks: Level[] = [];
			for (let i = 0; i < 30; i++) {
				bids.push({ price: mid - i * 8 - Math.random() * 4, size: Math.random() * 5 + 0.2 });
				asks.push({ price: mid + i * 8 + Math.random() * 4, size: Math.random() * 5 + 0.2 });
			}
			return { bids, asks };
		};
		let book = generateBook(livePrice);

		// Crosshair
		let mouseX = -1, mouseY = -1;
		chartCanvas.addEventListener('mousemove', (e) => {
			const r = chartCanvas.getBoundingClientRect();
			mouseX = e.clientX - r.left;
			mouseY = e.clientY - r.top;
		});
		chartCanvas.addEventListener('mouseleave', () => { mouseX = -1; mouseY = -1; });

		function getPriceRange() {
			const all = [...candles, liveCandle];
			const hi = Math.max(...all.map(c => c.high));
			const lo = Math.min(...all.map(c => c.low));
			const pad = (hi - lo) * 0.1;
			return { hi: hi + pad, lo: lo - pad };
		}

		function priceToY(price: number, hi: number, lo: number, h: number) {
			return h - ((price - lo) / (hi - lo)) * h * 0.85 - h * 0.06;
		}

		function drawChart() {
			cCtx.clearRect(0, 0, W, H);

			// Background
			const bg = cCtx.createLinearGradient(0, 0, 0, H);
			bg.addColorStop(0, '#040d18');
			bg.addColorStop(1, '#02070f');
			cCtx.fillStyle = bg;
			cCtx.fillRect(0, 0, W, H);

			const { hi, lo } = getPriceRange();
			const chartH = H * 0.72;
			const chartY = H * 0.06;
			const candleW = Math.min(14, (W * 0.82) / (CANDLE_COUNT + 2));
			const candleGap = candleW * 0.22;
			const totalW = (CANDLE_COUNT + 1) * (candleW + candleGap);
			const startX = W * 0.09;

			// Grid lines
			const gridCount = 6;
			for (let i = 0; i <= gridCount; i++) {
				const gPrice = lo + (hi - lo) * (i / gridCount);
				const gy = priceToY(gPrice, hi, lo, chartH) + chartY;
				cCtx.beginPath();
				cCtx.moveTo(startX, gy);
				cCtx.lineTo(W * 0.92, gy);
				cCtx.strokeStyle = 'rgba(40,80,140,0.18)';
				cCtx.lineWidth = 1;
				cCtx.stroke();
				// Price label
				cCtx.fillStyle = 'rgba(100,140,200,0.45)';
				cCtx.font = '10px var(--font-mono)';
				cCtx.textAlign = 'right';
				cCtx.fillText('$' + gPrice.toFixed(0), startX - 6, gy + 4);
			}

			// Volume bars
			const maxVol = Math.max(...candles.map(c => c.volume), liveCandle.volume);
			const volH = H * 0.12;
			const volY = H * 0.84;
			const allCandles = [...candles, liveCandle];

			for (let i = 0; i < allCandles.length; i++) {
				const c = allCandles[i];
				const x = startX + i * (candleW + candleGap);
				const vh = (c.volume / maxVol) * volH;
				const bull = c.close >= c.open;
				cCtx.fillStyle = bull ? 'rgba(0,210,100,0.2)' : 'rgba(255,60,80,0.2)';
				cCtx.fillRect(x, volY + volH - vh, candleW, vh);
			}

			// Candles
			for (let i = 0; i < allCandles.length; i++) {
				const c = allCandles[i];
				const x = startX + i * (candleW + candleGap);
				const isLive = i === allCandles.length - 1;
				const bull = c.close >= c.open;

				const openY = priceToY(c.open, hi, lo, chartH) + chartY;
				const closeY = priceToY(c.close, hi, lo, chartH) + chartY;
				const highY = priceToY(c.high, hi, lo, chartH) + chartY;
				const lowY = priceToY(c.low, hi, lo, chartH) + chartY;

				const bodyTop = Math.min(openY, closeY);
				const bodyH = Math.max(2, Math.abs(closeY - openY));

				// Wick
				cCtx.beginPath();
				cCtx.moveTo(x + candleW / 2, highY);
				cCtx.lineTo(x + candleW / 2, lowY);
				cCtx.strokeStyle = bull ? 'rgba(0,210,100,0.6)' : 'rgba(255,60,80,0.6)';
				cCtx.lineWidth = 1;
				cCtx.stroke();

				// Body
				if (isLive) {
					const pulse = 0.7 + Math.sin(time * 6) * 0.3;
					cCtx.shadowColor = bull ? `rgba(0,210,100,${pulse})` : `rgba(255,60,80,${pulse})`;
					cCtx.shadowBlur = 12;
				}
				cCtx.fillStyle = bull
					? (isLive ? `rgba(0,210,100,${0.85 + Math.sin(time * 4) * 0.15})` : 'rgba(0,190,85,0.85)')
					: (isLive ? `rgba(255,60,80,${0.85 + Math.sin(time * 4) * 0.15})` : 'rgba(240,50,70,0.85)');
				cCtx.fillRect(x, bodyTop, candleW, bodyH);
				cCtx.shadowBlur = 0;
			}

			// EMA lines
			const ema9: number[] = [];
			const ema21: number[] = [];
			const calcEMA = (arr: Candle[], period: number): number[] => {
				const k = 2 / (period + 1);
				const result: number[] = [];
				let prev = allCandles.slice(0, period).reduce((s, c) => s + c.close, 0) / period;
				for (let i = 0; i < arr.length; i++) {
					const val = i < period ? prev : arr[i].close * k + prev * (1 - k);
					result.push(val);
					prev = val;
				}
				return result;
			};
			const ema9vals = calcEMA(allCandles, 9);
			const ema21vals = calcEMA(allCandles, 21);

			for (const [vals, color] of [[ema9vals, 'rgba(255,180,0,0.7)'], [ema21vals, 'rgba(100,180,255,0.6)']] as [number[], string][]) {
				cCtx.beginPath();
				for (let i = 0; i < vals.length; i++) {
					const x = startX + i * (candleW + candleGap) + candleW / 2;
					const y = priceToY(vals[i], hi, lo, chartH) + chartY;
					i === 0 ? cCtx.moveTo(x, y) : cCtx.lineTo(x, y);
				}
				cCtx.strokeStyle = color;
				cCtx.lineWidth = 1.5;
				cCtx.stroke();
			}

			// Live price line
			const livePriceY = priceToY(livePrice, hi, lo, chartH) + chartY;
			cCtx.setLineDash([4, 4]);
			cCtx.beginPath();
			cCtx.moveTo(startX, livePriceY);
			cCtx.lineTo(W * 0.92, livePriceY);
			const bull = liveCandle.close >= liveCandle.open;
			cCtx.strokeStyle = bull ? 'rgba(0,210,100,0.5)' : 'rgba(255,60,80,0.5)';
			cCtx.lineWidth = 1;
			cCtx.stroke();
			cCtx.setLineDash([]);

			// Live price tag
			const tagColor = bull ? '#00d264' : '#ff3c50';
			cCtx.fillStyle = tagColor;
			cCtx.fillRect(W * 0.922, livePriceY - 10, 72, 20);
			cCtx.fillStyle = '#000';
			cCtx.font = 'bold 10px var(--font-mono)';
			cCtx.textAlign = 'left';
			cCtx.fillText('$' + livePrice.toFixed(1), W * 0.926, livePriceY + 4);

			// Crosshair
			if (mouseX >= 0 && mouseY >= 0) {
				cCtx.setLineDash([3, 3]);
				cCtx.strokeStyle = 'rgba(120,160,220,0.35)';
				cCtx.lineWidth = 1;
				cCtx.beginPath();
				cCtx.moveTo(mouseX, 0); cCtx.lineTo(mouseX, H);
				cCtx.moveTo(0, mouseY); cCtx.lineTo(W, mouseY);
				cCtx.stroke();
				cCtx.setLineDash([]);
				// Price at cursor
				const cursorPrice = lo + (hi - lo) * (1 - (mouseY - chartY) / chartH);
				cCtx.fillStyle = 'rgba(30,60,110,0.9)';
				cCtx.fillRect(0, mouseY - 10, startX - 8, 20);
				cCtx.fillStyle = 'rgba(140,180,255,0.9)';
				cCtx.font = '10px var(--font-mono)';
				cCtx.textAlign = 'right';
				cCtx.fillText('$' + cursorPrice.toFixed(0), startX - 10, mouseY + 4);
			}

			// Update live price tag DOM
			if (priceTagRef) {
				priceTagRef.textContent = '$' + livePrice.toFixed(2);
				priceTagRef.style.color = bull ? '#00d264' : '#ff3c50';
				priceTagRef.style.textShadow = bull
					? '0 0 20px rgba(0,210,100,0.6)' : '0 0 20px rgba(255,60,80,0.6)';
			}
		}

		function drawDepth() {
			const dW = depthCanvas.width;
			const dH = depthCanvas.height;
			dCtx.clearRect(0, 0, dW, dH);

			const dbg = dCtx.createLinearGradient(0, 0, 0, dH);
			dbg.addColorStop(0, '#030c18');
			dbg.addColorStop(1, '#020810');
			dCtx.fillStyle = dbg;
			dCtx.fillRect(0, 0, dW, dH);

			const { bids, asks } = book;
			const mid = livePrice;
			const span = 300;
			const priceMin = mid - span;
			const priceMax = mid + span;

			const maxSize = Math.max(...bids.map(b => b.size), ...asks.map(a => a.size));

			// Cumulative bid curve
			let cumBid = 0;
			dCtx.beginPath();
			dCtx.moveTo(0, dH);
			for (const b of [...bids].sort((a, v) => v.price - a.price)) {
				if (b.price < priceMin) continue;
				cumBid += b.size;
				const x = ((b.price - priceMin) / (priceMax - priceMin)) * dW;
				const y = dH - (cumBid / (maxSize * bids.length * 0.3)) * dH * 0.8;
				dCtx.lineTo(x, y);
			}
			dCtx.lineTo(dW * 0.5, dH);
			dCtx.closePath();
			const bidGrad = dCtx.createLinearGradient(0, 0, dW * 0.5, 0);
			bidGrad.addColorStop(0, 'rgba(0,210,100,0.03)');
			bidGrad.addColorStop(1, 'rgba(0,210,100,0.18)');
			dCtx.fillStyle = bidGrad;
			dCtx.fill();
			dCtx.strokeStyle = 'rgba(0,210,100,0.6)';
			dCtx.lineWidth = 1.5;
			dCtx.stroke();

			// Cumulative ask curve
			let cumAsk = 0;
			dCtx.beginPath();
			dCtx.moveTo(dW, dH);
			for (const a of [...asks].sort((a, v) => a.price - v.price)) {
				if (a.price > priceMax) continue;
				cumAsk += a.size;
				const x = ((a.price - priceMin) / (priceMax - priceMin)) * dW;
				const y = dH - (cumAsk / (maxSize * asks.length * 0.3)) * dH * 0.8;
				dCtx.lineTo(x, y);
			}
			dCtx.lineTo(dW * 0.5, dH);
			dCtx.closePath();
			const askGrad = dCtx.createLinearGradient(dW * 0.5, 0, dW, 0);
			askGrad.addColorStop(0, 'rgba(255,60,80,0.18)');
			askGrad.addColorStop(1, 'rgba(255,60,80,0.03)');
			dCtx.fillStyle = askGrad;
			dCtx.fill();
			dCtx.strokeStyle = 'rgba(255,60,80,0.6)';
			dCtx.lineWidth = 1.5;
			dCtx.stroke();

			// Mid line
			dCtx.beginPath();
			dCtx.moveTo(dW * 0.5, 0);
			dCtx.lineTo(dW * 0.5, dH);
			dCtx.strokeStyle = 'rgba(255,200,60,0.3)';
			dCtx.lineWidth = 1;
			dCtx.setLineDash([3, 3]);
			dCtx.stroke();
			dCtx.setLineDash([]);

			// Labels
			dCtx.fillStyle = 'rgba(0,210,100,0.55)';
			dCtx.font = '9px var(--font-mono)';
			dCtx.textAlign = 'left';
			dCtx.fillText('BIDS', 8, 14);
			dCtx.fillStyle = 'rgba(255,60,80,0.55)';
			dCtx.textAlign = 'right';
			dCtx.fillText('ASKS', dW - 8, 14);
			dCtx.fillStyle = 'rgba(255,200,60,0.5)';
			dCtx.textAlign = 'center';
			dCtx.fillText('DEPTH', dW / 2, 14);
		}

		// Tick simulation
		let tickInterval = setInterval(() => {
			const move = (Math.random() - 0.485) * 35;
			livePrice += move;
			liveCandle.close = livePrice;
			liveCandle.high = Math.max(liveCandle.high, livePrice);
			liveCandle.low = Math.min(liveCandle.low, livePrice);
			liveCandle.volume += 0.02;

			// Refresh book
			book = generateBook(livePrice);

			// Update ticker
			if (tickerRef) {
				const up = move > 0;
				tickerRef.style.color = up ? '#00d264' : '#ff3c50';
				gsap.fromTo(tickerRef, { y: up ? 6 : -6, opacity: 0 }, { y: 0, opacity: 1, duration: 0.25, ease: 'power2.out' });
			}

			// Occasionally close candle
			if (Math.random() > 0.97) {
				candles.push({ ...liveCandle });
				if (candles.length > CANDLE_COUNT) candles.shift();
				liveCandle = {
					open: livePrice, close: livePrice,
					high: livePrice + 10, low: livePrice - 10,
					volume: 0.05, timestamp: liveCandle.timestamp + 1,
				};
			}
		}, 200);

		function loop() {
			time += 0.016;
			drawChart();
			drawDepth();
			raf = requestAnimationFrame(loop);
		}
		loop();

		// Entrance
		gsap.set([titleRef, statsRef, ctaRef], { autoAlpha: 0, y: 30 });
		const tl = gsap.timeline({ delay: 0.3 });
		tl.to(titleRef, { autoAlpha: 1, y: 0, duration: 0.9, ease: 'expo.out' })
			.to(statsRef, { autoAlpha: 1, y: 0, duration: 0.8, ease: 'expo.out' }, '-=0.5')
			.to(ctaRef, { autoAlpha: 1, y: 0, duration: 0.7, ease: 'expo.out' }, '-=0.4');

		const onResize = () => {
			W = chartCanvas.width = depthCanvas.width = window.innerWidth;
			H = chartCanvas.height = window.innerHeight;
			depthCanvas.height = Math.floor(H * 0.28);
		};
		window.addEventListener('resize', onResize);

		return () => {
			cancelAnimationFrame(raf);
			clearInterval(tickInterval);
			window.removeEventListener('resize', onResize);
		};
	});
</script>

<section class="ht1" bind:this={section}>
	<canvas class="ht1__chart" bind:this={chartCanvas}></canvas>
	<canvas class="ht1__depth" bind:this={depthCanvas}></canvas>

	<div class="ht1__overlay">
		<div class="ht1__header" bind:this={titleRef}>
			<div class="ht1__symbol">
				<span class="ht1__sym-name">BTC/USDT</span>
				<span class="ht1__sym-ex">BINANCE · PERPETUAL</span>
			</div>
			<div class="ht1__live-price" bind:this={priceTagRef}>$42,380.00</div>
			<div class="ht1__tick" bind:this={tickerRef}>▲ +1.24%</div>
		</div>

		<div class="ht1__stats" bind:this={statsRef}>
			<div class="ht1__stat">
				<span class="ht1__stat-label">24H VOL</span>
				<span class="ht1__stat-val">$18.4B</span>
			</div>
			<div class="ht1__stat">
				<span class="ht1__stat-label">24H HIGH</span>
				<span class="ht1__stat-val ht1__bull">$43,812</span>
			</div>
			<div class="ht1__stat">
				<span class="ht1__stat-label">24H LOW</span>
				<span class="ht1__stat-val ht1__bear">$41,204</span>
			</div>
			<div class="ht1__stat">
				<span class="ht1__stat-label">OPEN INT</span>
				<span class="ht1__stat-val">$6.2B</span>
			</div>
			<div class="ht1__stat">
				<span class="ht1__stat-label">FUNDING</span>
				<span class="ht1__stat-val ht1__bull">+0.0102%</span>
			</div>
		</div>

		<div class="ht1__cta" bind:this={ctaRef}>
			<button class="ht1__btn ht1__btn-long">↑ LONG</button>
			<button class="ht1__btn ht1__btn-short">↓ SHORT</button>
			<span class="ht1__lev">25× Leverage</span>
		</div>
	</div>

	<div class="ht1__depth-label">ORDER BOOK DEPTH</div>
	<div class="ht1__vignette"></div>
</section>

<style>
	.ht1 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #020810;
		display: flex;
		align-items: flex-start;
	}

	.ht1__chart {
		position: absolute;
		inset: 0;
		width: 100%; height: 72%;
		z-index: 2;
		pointer-events: all;
	}

	.ht1__depth {
		position: absolute;
		bottom: 0; left: 0; right: 0;
		height: 28%;
		z-index: 2;
		pointer-events: none;
	}

	.ht1__overlay {
		position: relative;
		z-index: 10;
		padding: 2rem 2.5rem;
		display: flex;
		flex-direction: column;
		gap: 1rem;
		pointer-events: none;
	}

	.ht1__header {
		display: flex;
		align-items: center;
		gap: 1.5rem;
	}

	.ht1__symbol {
		display: flex;
		flex-direction: column;
	}

	.ht1__sym-name {
		font-family: var(--font-cinematic);
		font-size: 1.8rem;
		color: #fff;
		letter-spacing: 0.08em;
		line-height: 1;
	}

	.ht1__sym-ex {
		font-family: var(--font-mono);
		font-size: 0.6rem;
		color: rgba(100, 150, 220, 0.55);
		letter-spacing: 0.15em;
		margin-top: 2px;
	}

	.ht1__live-price {
		font-family: var(--font-mono);
		font-size: 1.6rem;
		font-weight: 700;
		color: #00d264;
		letter-spacing: 0.05em;
		text-shadow: 0 0 20px rgba(0, 210, 100, 0.4);
	}

	.ht1__tick {
		font-family: var(--font-mono);
		font-size: 0.85rem;
		color: #00d264;
		letter-spacing: 0.08em;
	}

	.ht1__stats {
		display: flex;
		gap: 2rem;
		flex-wrap: wrap;
	}

	.ht1__stat {
		display: flex;
		flex-direction: column;
		gap: 2px;
	}

	.ht1__stat-label {
		font-family: var(--font-mono);
		font-size: 0.58rem;
		color: rgba(100, 140, 200, 0.5);
		letter-spacing: 0.12em;
	}

	.ht1__stat-val {
		font-family: var(--font-mono);
		font-size: 0.82rem;
		color: rgba(200, 220, 255, 0.85);
		letter-spacing: 0.06em;
	}

	.ht1__bull { color: #00d264 !important; }
	.ht1__bear { color: #ff3c50 !important; }

	.ht1__cta {
		display: flex;
		align-items: center;
		gap: 0.8rem;
		pointer-events: all;
	}

	.ht1__btn {
		padding: 0.55rem 1.6rem;
		font-family: var(--font-mono);
		font-size: 0.8rem;
		font-weight: 700;
		letter-spacing: 0.12em;
		cursor: pointer;
		border: none;
		transition: all 0.2s;
	}

	.ht1__btn-long {
		background: rgba(0, 210, 100, 0.2);
		border: 1px solid rgba(0, 210, 100, 0.5);
		color: #00d264;
		box-shadow: 0 0 20px rgba(0, 210, 100, 0.15);
	}
	.ht1__btn-long:hover {
		background: rgba(0, 210, 100, 0.35);
		box-shadow: 0 0 40px rgba(0, 210, 100, 0.4);
		transform: translateY(-1px);
	}

	.ht1__btn-short {
		background: rgba(255, 60, 80, 0.2);
		border: 1px solid rgba(255, 60, 80, 0.5);
		color: #ff3c50;
		box-shadow: 0 0 20px rgba(255, 60, 80, 0.15);
	}
	.ht1__btn-short:hover {
		background: rgba(255, 60, 80, 0.35);
		box-shadow: 0 0 40px rgba(255, 60, 80, 0.4);
		transform: translateY(-1px);
	}

	.ht1__lev {
		font-family: var(--font-mono);
		font-size: 0.65rem;
		color: rgba(255, 200, 60, 0.55);
		border: 1px solid rgba(255, 200, 60, 0.2);
		padding: 0.3rem 0.6rem;
		letter-spacing: 0.08em;
	}

	.ht1__depth-label {
		position: absolute;
		bottom: calc(28% + 8px);
		left: 2.5rem;
		font-family: var(--font-mono);
		font-size: 0.58rem;
		letter-spacing: 0.2em;
		color: rgba(100, 140, 200, 0.35);
		z-index: 11;
	}

	.ht1__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at 50% 50%, transparent 50%, rgba(2,8,16,0.7) 100%);
		pointer-events: none;
		z-index: 6;
	}
</style>
