<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';

	let section: HTMLElement;
	let canvas: HTMLCanvasElement;
	let titleRef: HTMLElement;
	let ctaRef: HTMLElement;

	type Tile = {
		symbol: string; name: string; price: number; change: number;
		mcap: number; x: number; y: number; w: number; h: number;
		displayChange: number; flashTimer: number;
	};

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let W = (canvas.width = window.innerWidth);
		let H = (canvas.height = window.innerHeight);
		let raf: number;
		let time = 0;

		const ASSETS: Omit<Tile, 'x' | 'y' | 'w' | 'h' | 'displayChange' | 'flashTimer'>[] = [
			{ symbol: 'BTC',   name: 'Bitcoin',     price: 42380, change: 2.14,  mcap: 9 },
			{ symbol: 'ETH',   name: 'Ethereum',    price: 2280,  change: 3.55,  mcap: 7 },
			{ symbol: 'BNB',   name: 'BNB',         price: 398,   change: -1.22, mcap: 4 },
			{ symbol: 'SOL',   name: 'Solana',      price: 142,   change: 8.44,  mcap: 4 },
			{ symbol: 'XRP',   name: 'XRP',         price: 0.62,  change: -2.88, mcap: 3.5 },
			{ symbol: 'ADA',   name: 'Cardano',     price: 0.44,  change: 1.12,  mcap: 2.5 },
			{ symbol: 'AVAX',  name: 'Avalanche',   price: 38.2,  change: 5.67,  mcap: 2.2 },
			{ symbol: 'DOGE',  name: 'Dogecoin',    price: 0.088, change: -4.11, mcap: 2.0 },
			{ symbol: 'DOT',   name: 'Polkadot',    price: 7.22,  change: 0.88,  mcap: 1.8 },
			{ symbol: 'LINK',  name: 'Chainlink',   price: 14.8,  change: 6.33,  mcap: 1.6 },
			{ symbol: 'MATIC', name: 'Polygon',     price: 0.91,  change: -0.44, mcap: 1.4 },
			{ symbol: 'UNI',   name: 'Uniswap',     price: 6.55,  change: 2.77,  mcap: 1.2 },
			{ symbol: 'ATOM',  name: 'Cosmos',      price: 9.88,  change: -3.22, mcap: 1.0 },
			{ symbol: 'LTC',   name: 'Litecoin',    price: 74.2,  change: 1.55,  mcap: 0.9 },
			{ symbol: 'ARB',   name: 'Arbitrum',    price: 1.24,  change: 9.88,  mcap: 0.8 },
			{ symbol: 'OP',    name: 'Optimism',    price: 2.88,  change: 7.22,  mcap: 0.7 },
			{ symbol: 'INJ',   name: 'Injective',   price: 28.4,  change: 12.44, mcap: 0.7 },
			{ symbol: 'SUI',   name: 'Sui',         price: 1.44,  change: -5.88, mcap: 0.6 },
			{ symbol: 'APT',   name: 'Aptos',       price: 9.22,  change: 4.11,  mcap: 0.6 },
			{ symbol: 'FIL',   name: 'Filecoin',    price: 5.88,  change: -1.77, mcap: 0.5 },
			{ symbol: 'NEAR',  name: 'NEAR',        price: 3.44,  change: 3.55,  mcap: 0.5 },
			{ symbol: 'IMX',   name: 'Immutable',   price: 2.11,  change: 6.77,  mcap: 0.45 },
		];

		const tiles: Tile[] = ASSETS.map(a => ({
			...a, x: 0, y: 0, w: 0, h: 0,
			displayChange: a.change,
			flashTimer: 0,
		}));

		// Treemap layout (squarified)
		function layoutTreemap() {
			const topPad = H * 0.15;
			const botPad = H * 0.08;
			const sidePad = W * 0.02;
			const totalArea = (W - sidePad * 2) * (H - topPad - botPad);
			const totalMcap = tiles.reduce((s, t) => s + t.mcap, 0);

			let x = sidePad, y = topPad;
			let remW = W - sidePad * 2;
			let remH = H - topPad - botPad;
			let remaining = [...tiles];

			while (remaining.length > 0) {
				const row: Tile[] = [];
				let rowMcap = 0;
				const isHorizontal = remW >= remH;
				const stripLen = isHorizontal ? remH : remW;

				for (const t of remaining) {
					const testMcap = rowMcap + t.mcap;
					const testRatio = ((testMcap / totalMcap) * totalArea) / stripLen;
					if (row.length > 0) {
						const prevRatio = (rowMcap / totalMcap) * totalArea / stripLen;
						const newWorst = Math.max(testRatio, stripLen * stripLen / testRatio);
						const oldWorst = Math.max(prevRatio, stripLen * stripLen / prevRatio);
						if (newWorst > oldWorst) break;
					}
					row.push(t);
					rowMcap += t.mcap;
				}

				const stripW = isHorizontal ? (rowMcap / totalMcap) * totalArea / stripLen : remW;
				const stripH = isHorizontal ? remH : (rowMcap / totalMcap) * totalArea / stripLen;

				let cursor = isHorizontal ? y : x;
				for (const t of row) {
					const frac = t.mcap / rowMcap;
					if (isHorizontal) {
						t.x = x; t.y = cursor;
						t.w = stripW; t.h = stripH * frac;
						cursor += t.h;
					} else {
						t.x = cursor; t.y = y;
						t.w = stripW * frac; t.h = stripH;
						cursor += t.w;
					}
				}
				remaining = remaining.filter(t => !row.includes(t));
				if (isHorizontal) { x += stripW; remW -= stripW; }
				else { y += stripH; remH -= stripH; }
			}
		}
		layoutTreemap();

		// Live price ticks
		tiles.forEach(t => {
			setInterval(() => {
				const move = (Math.random() - 0.5) * Math.abs(t.change) * 0.3;
				t.price *= (1 + move * 0.001);
				t.displayChange += move * 0.1;
				t.flashTimer = 1.0;
			}, 400 + Math.random() * 600);
		});

		function tileColor(change: number, alpha = 1): string {
			const intensity = Math.min(1, Math.abs(change) / 12);
			if (change > 0) {
				const g = Math.floor(100 + intensity * 110);
				return `rgba(0,${g},${Math.floor(intensity * 50)},${alpha})`;
			} else {
				const r = Math.floor(160 + intensity * 95);
				return `rgba(${r},${Math.floor(40 - intensity * 20)},${Math.floor(40 + intensity * 20)},${alpha})`;
			}
		}

		function drawTile(t: Tile) {
			const { x, y, w, h } = t;
			const bull = t.displayChange >= 0;
			const intensity = Math.min(1, Math.abs(t.displayChange) / 12);
			const gap = 2;

			// Background
			ctx.fillStyle = tileColor(t.displayChange, 0.5 + intensity * 0.35);
			ctx.beginPath();
			ctx.roundRect(x + gap, y + gap, w - gap * 2, h - gap * 2, 3);
			ctx.fill();

			// Flash on tick
			if (t.flashTimer > 0) {
				t.flashTimer -= 0.06;
				ctx.fillStyle = bull
					? `rgba(0,255,120,${t.flashTimer * 0.25})`
					: `rgba(255,80,80,${t.flashTimer * 0.25})`;
				ctx.beginPath();
				ctx.roundRect(x + gap, y + gap, w - gap * 2, h - gap * 2, 3);
				ctx.fill();
			}

			// Border
			ctx.strokeStyle = bull
				? `rgba(0,210,100,${0.15 + intensity * 0.25})`
				: `rgba(255,60,80,${0.15 + intensity * 0.25})`;
			ctx.lineWidth = 1;
			ctx.stroke();

			// Content — only if tile is large enough
			if (w < 40 || h < 30) return;

			ctx.save();
			ctx.beginPath();
			ctx.roundRect(x + gap, y + gap, w - gap * 2, h - gap * 2, 3);
			ctx.clip();

			const fontSize = Math.min(18, Math.max(9, w * 0.14));
			ctx.fillStyle = 'rgba(255,255,255,0.92)';
			ctx.font = `bold ${fontSize}px var(--font-mono)`;
			ctx.textAlign = 'center';
			ctx.textBaseline = 'middle';

			if (h > 55) {
				ctx.fillText(t.symbol, x + w / 2, y + h * 0.38);
				const pctSize = Math.min(13, Math.max(8, w * 0.1));
				ctx.font = `${pctSize}px var(--font-mono)`;
				ctx.fillStyle = bull ? 'rgba(180,255,200,0.9)' : 'rgba(255,160,160,0.9)';
				const sign = bull ? '+' : '';
				ctx.fillText(sign + t.displayChange.toFixed(2) + '%', x + w / 2, y + h * 0.62);
			} else {
				ctx.fillText(t.symbol, x + w / 2, y + h / 2);
			}

			ctx.restore();
		}

		function draw() {
			ctx.clearRect(0, 0, W, H);
			time += 0.016;

			// Deep space bg
			ctx.fillStyle = '#030810';
			ctx.fillRect(0, 0, W, H);

			// Draw tiles
			tiles.forEach(t => drawTile(t));

			// Scan line sweep
			const scanGrad = ctx.createLinearGradient(0, 0, 0, H);
			const scanY = (Math.sin(time * 0.4) * 0.5 + 0.5) * H;
			scanGrad.addColorStop(Math.max(0, (scanY - 60) / H), 'rgba(60,140,255,0)');
			scanGrad.addColorStop(scanY / H, 'rgba(60,140,255,0.04)');
			scanGrad.addColorStop(Math.min(1, (scanY + 60) / H), 'rgba(60,140,255,0)');
			ctx.fillStyle = scanGrad;
			ctx.fillRect(0, 0, W, H);

			raf = requestAnimationFrame(draw);
		}
		draw();

		// Entrance: tiles fly in from random directions
		gsap.set([titleRef, ctaRef], { autoAlpha: 0 });
		tiles.forEach((t, i) => {
			const origX = t.x, origY = t.y;
			t.x = origX + (Math.random() - 0.5) * W * 0.4;
			t.y = origY + (Math.random() - 0.5) * H * 0.4;
			gsap.to(t, {
				x: origX, y: origY,
				duration: 0.9 + Math.random() * 0.4,
				delay: 0.1 + i * 0.018,
				ease: 'expo.out',
			});
		});
		gsap.to(titleRef, { autoAlpha: 1, duration: 0.8, delay: 0.3, ease: 'power2.out' });
		gsap.to(ctaRef, { autoAlpha: 1, duration: 0.7, delay: 0.8, ease: 'power2.out' });

		const onResize = () => {
			W = canvas.width = window.innerWidth;
			H = canvas.height = window.innerHeight;
			layoutTreemap();
		};
		window.addEventListener('resize', onResize);

		return () => {
			cancelAnimationFrame(raf);
			window.removeEventListener('resize', onResize);
		};
	});
</script>

<section class="ht3" bind:this={section}>
	<canvas bind:this={canvas}></canvas>

	<div class="ht3__header" bind:this={titleRef}>
		<div class="ht3__title">
			<span class="ht3__title-main">MARKET</span>
			<span class="ht3__title-sub">HEATMAP</span>
		</div>
		<div class="ht3__legend">
			<span class="ht3__leg-item ht3__bull">▲ Gaining</span>
			<span class="ht3__leg-item ht3__bear">▼ Losing</span>
			<span class="ht3__leg-size">Size = Market Cap</span>
		</div>
	</div>

	<div class="ht3__cta" bind:this={ctaRef}>
		<button class="ht3__btn">Full Heatmap</button>
	</div>

	<div class="ht3__vignette"></div>
</section>

<style>
	.ht3 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #030810;
	}

	canvas {
		position: absolute;
		inset: 0;
		width: 100%; height: 100%;
		z-index: 2;
	}

	.ht3__header {
		position: absolute;
		top: 0; left: 0; right: 0;
		height: 14vh;
		z-index: 10;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 2.5rem;
		background: rgba(3,8,16,0.9);
		border-bottom: 1px solid rgba(40,80,140,0.25);
		pointer-events: none;
	}

	.ht3__title {
		display: flex;
		align-items: baseline;
		gap: 0.6rem;
	}

	.ht3__title-main {
		font-family: var(--font-cinematic);
		font-size: clamp(1.5rem, 3.5vw, 3rem);
		letter-spacing: 0.12em;
		color: #fff;
	}

	.ht3__title-sub {
		font-family: var(--font-cinematic);
		font-size: clamp(1rem, 2.5vw, 2rem);
		letter-spacing: 0.18em;
		color: rgba(100,160,255,0.6);
	}

	.ht3__legend {
		display: flex;
		gap: 1.2rem;
		align-items: center;
		font-family: var(--font-mono);
		font-size: 0.68rem;
		letter-spacing: 0.1em;
	}

	.ht3__bull { color: rgba(0,210,100,0.8); }
	.ht3__bear { color: rgba(255,60,80,0.8); }
	.ht3__leg-size { color: rgba(120,160,220,0.45); }

	.ht3__cta {
		position: absolute;
		bottom: 2.5rem;
		right: 2.5rem;
		z-index: 10;
	}

	.ht3__btn {
		padding: 0.6rem 1.6rem;
		background: rgba(60,120,255,0.15);
		border: 1px solid rgba(80,150,255,0.35);
		color: rgba(160,200,255,0.9);
		font-family: var(--font-mono);
		font-size: 0.72rem;
		letter-spacing: 0.12em;
		cursor: pointer;
		transition: all 0.2s;
	}

	.ht3__btn:hover {
		background: rgba(60,120,255,0.3);
		box-shadow: 0 0 20px rgba(80,150,255,0.3);
	}

	.ht3__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at center, transparent 55%, rgba(3,8,16,0.6) 100%);
		pointer-events: none;
		z-index: 5;
	}
</style>
