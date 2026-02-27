<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	let section: HTMLElement;
	let gridCanvas: HTMLCanvasElement;
	let candleCanvas: HTMLCanvasElement;
	let wrThreat: HTMLElement;
	let wrMainTitle: HTMLElement;
	let wrMainSub: HTMLElement;
	let wrPriceTicker: HTMLElement;
	let wrRadar: HTMLElement;
	let wrAlertBar: HTMLElement;
	let wrMiniContainer: HTMLElement;
	let leftStream: HTMLElement;
	let rightStream: HTMLElement;
	let scanlineEl: HTMLElement;

	onMount(() => {
		gsap.registerPlugin(ScrollTrigger);

		// ── Grid canvas ──
		const gridCtx = gridCanvas.getContext('2d')!;
		let gw = (gridCanvas.width = window.innerWidth);
		let gh = (gridCanvas.height = window.innerHeight);
		let gridRaf: number;

		function resizeGrid() { gw = gridCanvas.width = window.innerWidth; gh = gridCanvas.height = window.innerHeight; }
		window.addEventListener('resize', resizeGrid);

		const gridAlpha = { val: 0 };
		let pulsePhase = 0;

		function drawGrid() {
			gridCtx.clearRect(0, 0, gw, gh);
			if (gridAlpha.val > 0) {
				pulsePhase += 0.005;
				const spacing = 50, cx = gw / 2, cy = gh / 2;
				gridCtx.strokeStyle = `rgba(0,255,136,${0.04 * gridAlpha.val})`;
				gridCtx.lineWidth = 0.5;
				for (let y = 0; y < gh; y += spacing) {
					const wave = Math.sin(pulsePhase + y * 0.01) * 3 * (Math.abs(y - cy) / (gh / 2));
					gridCtx.beginPath(); gridCtx.moveTo(0, y + wave); gridCtx.lineTo(gw, y + wave); gridCtx.stroke();
				}
				for (let x = 0; x < gw; x += spacing) {
					const wave = Math.sin(pulsePhase + x * 0.01) * 3 * (Math.abs(x - cx) / (gw / 2));
					gridCtx.beginPath(); gridCtx.moveTo(x + wave, 0); gridCtx.lineTo(x + wave, gh); gridCtx.stroke();
				}
				gridCtx.strokeStyle = `rgba(0,255,136,${0.15 * gridAlpha.val})`;
				gridCtx.lineWidth = 1; gridCtx.setLineDash([8, 8]);
				gridCtx.beginPath(); gridCtx.moveTo(cx, 0); gridCtx.lineTo(cx, gh); gridCtx.stroke();
				gridCtx.beginPath(); gridCtx.moveTo(0, cy); gridCtx.lineTo(gw, cy); gridCtx.stroke();
				gridCtx.setLineDash([]);
				const pr = ((pulsePhase * 80) % (Math.max(gw, gh) * 0.8));
				gridCtx.beginPath(); gridCtx.arc(cx, cy, pr, 0, Math.PI * 2);
				gridCtx.strokeStyle = `rgba(0,255,136,${(1 - pr / (Math.max(gw, gh) * 0.8)) * 0.1 * gridAlpha.val})`;
				gridCtx.lineWidth = 1; gridCtx.stroke();
			}
			gridRaf = requestAnimationFrame(drawGrid);
		}
		drawGrid();

		// ── Mini candle canvas ──
		const cctx = candleCanvas.getContext('2d')!;
		candleCanvas.width = 200; candleCanvas.height = 250;
		const candleAlpha = { val: 0 };

		type Candle = { open: number; close: number; high: number; low: number; bull: boolean };
		let candles: Candle[] = [];
		function generateCandles() {
			candles = [];
			let price = 100;
			for (let i = 0; i < 30; i++) {
				const open = price;
				const close = price + (Math.random() - 0.45) * 8;
				const high = Math.max(open, close) + Math.random() * 4;
				const low = Math.min(open, close) - Math.random() * 4;
				candles.push({ open, close, high, low, bull: close > open });
				price = close;
			}
		}
		generateCandles();

		function drawCandles() {
			cctx.clearRect(0, 0, 200, 250);
			if (candleAlpha.val > 0) {
				const minP = Math.min(...candles.map(c => c.low));
				const maxP = Math.max(...candles.map(c => c.high));
				const range = maxP - minP, cw = 5, gap = 1.5;
				candles.forEach((c, i) => {
					const x = i * (cw + gap);
					const oY = 250 - ((c.open - minP) / range) * 230 - 10;
					const cY = 250 - ((c.close - minP) / range) * 230 - 10;
					const hY = 250 - ((c.high - minP) / range) * 230 - 10;
					const lY = 250 - ((c.low - minP) / range) * 230 - 10;
					cctx.globalAlpha = candleAlpha.val * 0.7;
					cctx.strokeStyle = c.bull ? 'rgba(0,255,136,0.5)' : 'rgba(255,51,51,0.5)';
					cctx.lineWidth = 1;
					cctx.beginPath(); cctx.moveTo(x + cw / 2, hY); cctx.lineTo(x + cw / 2, lY); cctx.stroke();
					cctx.fillStyle = c.bull ? 'rgba(0,255,136,0.6)' : 'rgba(255,51,51,0.6)';
					cctx.fillRect(x, Math.min(oY, cY), cw, Math.abs(cY - oY) || 1);
				});
				cctx.globalAlpha = 1;
			}
			requestAnimationFrame(drawCandles);
		}
		drawCandles();

		// ── Data streams ──
		function generateDataStream(el: HTMLElement) {
			const symbols = ['ES', 'NQ', 'SPX', 'VIX', 'AAPL', 'TSLA', 'NVDA', 'AMZN'];
			el.textContent = Array.from({ length: 60 }, () => {
				const sym = symbols[Math.floor(Math.random() * symbols.length)];
				const price = (Math.random() * 500 + 100).toFixed(2);
				const change = ((Math.random() - 0.45) * 10).toFixed(2);
				return `${sym} ${price} ${parseFloat(change) >= 0 ? '+' : ''}${change}`;
			}).join('\n');
		}
		generateDataStream(leftStream);
		generateDataStream(rightStream);

		function scrollStream(el: HTMLElement, speed: number) {
			let y = 0;
			(function tick() {
				y -= speed;
				if (y < -(el.scrollHeight / 2)) y = 0;
				el.style.transform = `translateY(${y}px)`;
				requestAnimationFrame(tick);
			})();
		}

		// ── Title chars ──
		const wrText = 'COMMAND';
		wrMainTitle.innerHTML = wrText.split('').map(c => `<span class="wr-title-char">${c}</span>`).join('');

		function screenShake(target: Element | string, intensity = 8, duration = 0.3) {
			const tl = gsap.timeline();
			const steps = Math.floor(duration / 0.03);
			for (let i = 0; i < steps; i++) {
				const d = 1 - i / steps;
				tl.set(target, { x: (Math.random() - 0.5) * intensity * d, y: (Math.random() - 0.5) * intensity * d }, i * 0.03);
			}
			tl.set(target, { x: 0, y: 0 }); return tl;
		}

		// ── Radar sweep ──
		gsap.to(section.querySelector('.wr-radar-sweep'), { rotation: 360, duration: 3, ease: 'none', repeat: -1, transformOrigin: 'left center' });

		// ── Typing cursor blink ──
		gsap.to(section.querySelector('.wr-typing-cursor'), { opacity: 0, duration: 0.4, repeat: -1, yoyo: true, ease: 'steps(1)' });

		// ── MASTER TIMELINE ──
		const wrTl = gsap.timeline({
			scrollTrigger: { trigger: section, start: 'top 70%', toggleActions: 'play none none none' }
		});

		wrTl.to(gridAlpha, { val: 1, duration: 2, ease: 'power2.out' });
		wrTl.to(wrAlertBar, { opacity: 1, scaleX: 1, duration: 0.3, ease: 'power4.out' }, 0.5);
		wrTl.to(wrAlertBar, { opacity: 0, duration: 0.3 }, 1);
		wrTl.to(wrAlertBar, { opacity: 1, duration: 0.15 }, 1.5);
		wrTl.to(wrAlertBar, { opacity: 0, duration: 0.5 }, 1.8);

		wrTl.to(section.querySelectorAll('.wr-corner-bracket'), { opacity: 1, duration: 0.3, stagger: 0.08 }, 0.8);
		wrTl.to(scanlineEl, { top: '100%', duration: 2, ease: 'none', repeat: -1 }, 0);

		wrTl.to(wrThreat, { opacity: 1, duration: 0.3 }, 1.5);
		wrTl.add(() => {
			const blink = section.querySelector<HTMLElement>('.wr-blink');
			if (blink) blink.style.animation = 'wrBlink 0.5s infinite';
		}, 1.5);

		wrTl.to(section.querySelectorAll('.wr-title-char'), {
			opacity: 1, y: 0, duration: 0.08, stagger: 0.06, ease: 'none',
			onStart() {
				gsap.utils.toArray<HTMLElement>('.wr-title-char').forEach((ch, i) => {
					gsap.delayedCall(i * 0.06, () => gsap.fromTo(ch, { x: (Math.random() - 0.5) * 20 }, { x: 0, duration: 0.15 }));
				});
			}
		}, 2);

		wrTl.add(screenShake(section.querySelector('.wr-center')!, 8, 0.3), 2.5);
		wrTl.to(wrMainSub, { opacity: 1, duration: 0.5 }, 2.8);
		wrTl.to([leftStream, rightStream], { opacity: 1, duration: 1 }, 1.5);
		wrTl.add(() => { scrollStream(leftStream, 0.5); scrollStream(rightStream, 0.7); }, 1.5);
		wrTl.to(wrRadar, { opacity: 1, duration: 0.8 }, 2);
		wrTl.to(section.querySelectorAll('.wr-radar-dot'), {
			opacity: 1, duration: 0.2, stagger: 0.3,
			onComplete() {
				gsap.to(section.querySelectorAll('.wr-radar-dot'), { scale: 1.5, opacity: 0.5, duration: 0.8, repeat: -1, yoyo: true, stagger: 0.2 });
			}
		}, 2.5);

		wrTl.to(wrMiniContainer, { opacity: 1, duration: 1 }, 2);
		wrTl.to(candleAlpha, { val: 1, duration: 1 }, 2);
		wrTl.to(wrPriceTicker, { opacity: 1, duration: 0.8 }, 3);

		setInterval(() => {
			section.querySelectorAll<HTMLElement>('.wr-price').forEach(el => {
				const base = parseFloat(el.textContent!.replace(',', ''));
				const change = (Math.random() - 0.48) * 5;
				el.textContent = (base + change).toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
			});
		}, 2000);

		return () => {
			cancelAnimationFrame(gridRaf);
			window.removeEventListener('resize', resizeGrid);
		};
	});
</script>

<section class="wr-section" bind:this={section}>
	<canvas class="wr-grid-canvas" bind:this={gridCanvas}></canvas>

	<div class="wr-hud">
		<div class="wr-corner-bracket tl"></div>
		<div class="wr-corner-bracket tr"></div>
		<div class="wr-corner-bracket bl"></div>
		<div class="wr-corner-bracket br"></div>
	</div>

	<div class="wr-scanline" bind:this={scanlineEl}></div>
	<div class="wr-alert-bar" bind:this={wrAlertBar}></div>
	<div class="wr-data-stream wr-left" bind:this={leftStream}></div>
	<div class="wr-data-stream wr-right" bind:this={rightStream}></div>

	<div class="wr-radar" bind:this={wrRadar}>
		<div class="wr-radar-ring r1"></div>
		<div class="wr-radar-ring r2"></div>
		<div class="wr-radar-sweep"></div>
		<div class="wr-radar-dot" style="top:30%;left:60%"></div>
		<div class="wr-radar-dot" style="top:65%;left:35%"></div>
		<div class="wr-radar-dot" style="top:45%;left:70%"></div>
	</div>

	<div class="wr-minicandle-container" bind:this={wrMiniContainer}>
		<canvas bind:this={candleCanvas}></canvas>
	</div>

	<div class="wr-center">
		<div class="wr-threat" bind:this={wrThreat}>⚠ THREAT LEVEL: <span class="wr-blink">MAXIMUM</span></div>
		<h1 class="wr-main-title" aria-label="COMMAND" bind:this={wrMainTitle}></h1>
		<div class="wr-main-sub" bind:this={wrMainSub}>EXECUTING PROTOCOL...<span class="wr-typing-cursor"></span></div>
	</div>

	<div class="wr-price-ticker" bind:this={wrPriceTicker}>
		<div class="wr-ticker-item"><span class="wr-symbol">SPX</span><span class="wr-price">5,892.47</span><span class="wr-change up">+2.31%</span></div>
		<div class="wr-ticker-item"><span class="wr-symbol">NQ</span><span class="wr-price">21,340.12</span><span class="wr-change up">+1.87%</span></div>
		<div class="wr-ticker-item"><span class="wr-symbol">VIX</span><span class="wr-price">18.42</span><span class="wr-change down">-12.4%</span></div>
		<div class="wr-ticker-item"><span class="wr-symbol">ES</span><span class="wr-price">5,901.25</span><span class="wr-change up">+0.94%</span></div>
	</div>
</section>

<style>
	@keyframes wrBlink { 0%,100%{opacity:1}50%{opacity:0} }

	.wr-section { position: relative; height: 100vh; overflow: hidden; background: #020508; }
	.wr-grid-canvas { position: absolute; inset: 0; z-index: 1; width: 100%; height: 100%; }
	.wr-hud { position: absolute; inset: 0; z-index: 5; pointer-events: none; }

	.wr-corner-bracket { position: absolute; width: 80px; height: 80px; opacity: 0; }
	.wr-corner-bracket::before, .wr-corner-bracket::after { content: ''; position: absolute; background: #00ff88; }
	.wr-corner-bracket.tl::before { top: 0; left: 0; width: 30px; height: 1px; }
	.wr-corner-bracket.tl::after  { top: 0; left: 0; width: 1px;  height: 30px; }
	.wr-corner-bracket.tr::before { top: 0; right: 0; width: 30px; height: 1px; }
	.wr-corner-bracket.tr::after  { top: 0; right: 0; width: 1px;  height: 30px; }
	.wr-corner-bracket.bl::before { bottom: 0; left: 0; width: 30px; height: 1px; }
	.wr-corner-bracket.bl::after  { bottom: 0; left: 0; width: 1px; height: 30px; }
	.wr-corner-bracket.br::before { bottom: 0; right: 0; width: 30px; height: 1px; }
	.wr-corner-bracket.br::after  { bottom: 0; right: 0; width: 1px; height: 30px; }
	.wr-corner-bracket.tl { top: 30px; left: 30px; }
	.wr-corner-bracket.tr { top: 30px; right: 30px; }
	.wr-corner-bracket.bl { bottom: 30px; left: 30px; }
	.wr-corner-bracket.br { bottom: 30px; right: 30px; }

	.wr-scanline { position: absolute; left: 0; right: 0; height: 2px; background: linear-gradient(90deg, transparent, rgba(0,255,136,0.15), transparent); z-index: 6; top: -2px; pointer-events: none; }
	.wr-alert-bar { position: absolute; top: 0; left: 0; right: 0; height: 2px; background: linear-gradient(90deg, transparent, #ff3333, transparent); z-index: 20; opacity: 0; transform: scaleX(0); }
	.wr-data-stream { position: absolute; z-index: 4; font-family: 'JetBrains Mono', monospace; font-size: 0.55rem; color: rgba(0,255,136,0.25); letter-spacing: 0.1em; line-height: 1.4; white-space: pre; overflow: hidden; opacity: 0; }
	.wr-left  { top: 15%; left: 30px; width: 180px; height: 60%; text-align: left; }
	.wr-right { top: 10%; right: 30px; width: 180px; height: 70%; text-align: right; }

	.wr-radar { position: absolute; bottom: 60px; left: 60px; width: 160px; height: 160px; z-index: 5; border: 1px solid rgba(0,255,136,0.15); border-radius: 50%; opacity: 0; }
	.wr-radar-sweep { position: absolute; top: 50%; left: 50%; width: 50%; height: 2px; background: linear-gradient(90deg, rgba(0,255,136,0.6), transparent); transform-origin: left center; }
	.wr-radar-ring { position: absolute; border: 1px solid rgba(0,255,136,0.08); border-radius: 50%; }
	.wr-radar-ring.r1 { inset: 20%; } .wr-radar-ring.r2 { inset: 40%; }
	.wr-radar-dot { position: absolute; width: 4px; height: 4px; border-radius: 50%; background: #00ff88; opacity: 0; box-shadow: 0 0 8px #00ff88; }

	.wr-minicandle-container { position: absolute; top: 15%; right: 220px; width: 200px; height: 250px; z-index: 4; opacity: 0; }
	.wr-minicandle-container canvas { width: 100%; height: 100%; }

	.wr-center { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); z-index: 10; text-align: center; }
	.wr-threat { font-family: 'Orbitron', sans-serif; font-size: 0.65rem; letter-spacing: 0.5em; color: #ff3333; margin-bottom: 15px; opacity: 0; }
	.wr-main-title { font-family: 'Bebas Neue', sans-serif; font-size: clamp(4rem, 12vw, 10rem); line-height: 0.85; color: #fff; letter-spacing: 0.05em; }
	:global(.wr-title-char) { display: inline-block; opacity: 0; transform: translateY(80px); text-shadow: 0 0 40px rgba(0,255,136,0.3); }
	.wr-main-sub { font-family: 'JetBrains Mono', monospace; font-size: 0.75rem; letter-spacing: 0.25em; color: rgba(0,255,136,0.5); margin-top: 20px; opacity: 0; }
	.wr-typing-cursor { display: inline-block; width: 8px; height: 14px; background: #00ff88; margin-left: 4px; vertical-align: middle; }

	.wr-price-ticker { position: absolute; bottom: 30px; left: 50%; transform: translateX(-50%); z-index: 10; display: flex; gap: 40px; opacity: 0; }
	.wr-ticker-item { font-family: 'JetBrains Mono', monospace; font-size: 0.65rem; letter-spacing: 0.15em; }
	.wr-symbol { color: rgba(255,255,255,0.5); }
	.wr-price  { color: #fff; margin: 0 6px; }
	.wr-change.up   { color: #00ff88; }
	.wr-change.down { color: #ff3333; }
</style>
