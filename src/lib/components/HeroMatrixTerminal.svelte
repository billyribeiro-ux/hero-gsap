<script context="module" lang="ts">
	class Particle {
		x: number; y: number; baseX: number; baseY: number;
		vx: number; vy: number; size: number; alpha: number;
		pulse: number; pulseSpeed: number; depth: number;
		constructor(w: number, h: number) {
			this.x = Math.random() * w; this.y = Math.random() * h;
			this.baseX = this.x; this.baseY = this.y;
			this.vx = (Math.random() - 0.5) * 0.5; this.vy = (Math.random() - 0.5) * 0.5;
			this.size = Math.random() * 2 + 0.5;
			this.alpha = Math.random() * 0.5 + 0.1;
			this.pulse = Math.random() * Math.PI * 2;
			this.pulseSpeed = 0.02 + Math.random() * 0.03;
			this.depth = Math.random();
		}
		update(mouseX: number, mouseY: number, w: number, h: number) {
			this.pulse += this.pulseSpeed;
			const dx = mouseX - this.x; const dy = mouseY - this.y;
			const dist = Math.sqrt(dx * dx + dy * dy);
			if (dist < 200) {
				const force = (1 - dist / 200) * 2;
				this.vx += (dx / dist) * force * 0.1;
				this.vy += (dy / dist) * force * 0.1;
			}
			this.vx += (this.baseX - this.x) * 0.001;
			this.vy += (this.baseY - this.y) * 0.001;
			this.vx *= 0.98; this.vy *= 0.98;
			this.x += this.vx; this.y += this.vy;
			if (this.x < -50) this.x = w + 50;
			if (this.x > w + 50) this.x = -50;
			if (this.y < -50) this.y = h + 50;
			if (this.y > h + 50) this.y = -50;
		}
	}
</script>

<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';

	let section: HTMLElement;
	let matrixCanvas: HTMLCanvasElement;
	let trailCanvas: HTMLCanvasElement;
	let particleCanvas: HTMLCanvasElement;
	let gridCanvas: HTMLCanvasElement;
	let chartCanvas: HTMLCanvasElement;
	let hudCanvas: HTMLCanvasElement;
	let cursorEl: HTMLElement;
	let cursorDot: HTMLElement;
	let bootScreen: HTMLElement;
	let topBar: HTMLElement;
	let centerTitle: HTMLElement;
	let centerSub: HTMLElement;
	let centerTagline: HTMLElement;
	let panelOrderbook: HTMLElement;
	let panelPositions: HTMLElement;
	let tickerTape: HTMLElement;
	let tickerInner: HTMLElement;
	let orderbookContent: HTMLElement;
	let positionsContent: HTMLElement;
	let clockEl: HTMLElement;
	let fpsEl: HTMLElement;
	let scanlineEl: HTMLElement;
	let alertFlash: HTMLElement;
	let glitchSlice1: HTMLElement;
	let glitchSlice2: HTMLElement;

	onMount(() => {
		let mouseX = window.innerWidth / 2;
		let mouseY = window.innerHeight / 2;
		let frameCount = 0;
		let lastTime = performance.now();
		let systemOnline = false;
		let rafIds: number[] = [];

		const W = () => window.innerWidth;
		const H = () => window.innerHeight;

		// Canvas setup
		const ctxMatrix = matrixCanvas.getContext('2d')!;
		const ctxTrail = trailCanvas.getContext('2d')!;
		const ctxParticle = particleCanvas.getContext('2d')!;
		const ctxGrid = gridCanvas.getContext('2d')!;
		const ctxChart = chartCanvas.getContext('2d')!;
		const ctxHud = hudCanvas.getContext('2d')!;

		function resizeAll() {
			[matrixCanvas, trailCanvas, particleCanvas, gridCanvas, chartCanvas, hudCanvas].forEach(c => {
				c.width = W(); c.height = H();
			});
		}
		resizeAll();
		window.addEventListener('resize', resizeAll);

		// Mouse
		const trailPoints: { x: number; y: number; life: number }[] = [];
		document.addEventListener('mousemove', e => {
			mouseX = e.clientX; mouseY = e.clientY;
			trailPoints.push({ x: mouseX, y: mouseY, life: 1 });
			if (trailPoints.length > 60) trailPoints.shift();
		});

		// Custom cursor
		function updateCursor() {
			gsap.to(cursorEl, { x: mouseX - 10, y: mouseY - 10, duration: 0.15, ease: 'power2.out' });
			gsap.set(cursorDot, { x: mouseX - 2, y: mouseY - 2 });
			rafIds.push(requestAnimationFrame(updateCursor));
		}
		updateCursor();

		/* ── MATRIX RAIN ── */
		const CHARS = 'アイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホマミムメモヤユヨラリルレロワヲン0123456789ABCDEF$¥€£∞∑∏∂∫';
		const charArr = CHARS.split('');
		const fontSize = 14;
		let columns = 0;
		let drops: number[] = [];
		let columnSpeeds: number[] = [];
		let columnBrightness: number[] = [];
		let columnChars: string[][] = [];

		function initMatrix() {
			columns = Math.ceil(W() / fontSize);
			drops = []; columnSpeeds = []; columnBrightness = []; columnChars = [];
			for (let i = 0; i < columns; i++) {
				drops[i] = Math.random() * -100;
				columnSpeeds[i] = 0.3 + Math.random() * 0.8;
				columnBrightness[i] = 0.3 + Math.random() * 0.7;
				columnChars[i] = [];
				const colH = Math.ceil(H() / fontSize);
				for (let j = 0; j < colH; j++) {
					columnChars[i][j] = charArr[Math.floor(Math.random() * charArr.length)];
				}
			}
		}
		initMatrix();
		window.addEventListener('resize', initMatrix);

		function drawMatrix() {
			ctxMatrix.fillStyle = 'rgba(0,0,0,0.05)';
			ctxMatrix.fillRect(0, 0, W(), H());
			ctxMatrix.font = `${fontSize}px 'JetBrains Mono', monospace`;
			for (let i = 0; i < columns; i++) {
				const x = i * fontSize;
				const dropRow = Math.floor(drops[i]);
				const colH = Math.ceil(H() / fontSize);
				const dx = x - mouseX;
				const dy = dropRow * fontSize - mouseY;
				const dist = Math.sqrt(dx * dx + dy * dy);
				const mouseFactor = Math.max(0, 1 - dist / 300) * 0.5;
				const y = dropRow * fontSize;
				if (y > 0 && y < H()) {
					if (Math.random() > 0.97) columnChars[i][dropRow % colH] = charArr[Math.floor(Math.random() * charArr.length)];
					const brightness = columnBrightness[i] + mouseFactor;
					ctxMatrix.fillStyle = `rgba(180,255,220,${Math.min(1, brightness)})`;
					ctxMatrix.shadowBlur = 15; ctxMatrix.shadowColor = '#00ff88';
					ctxMatrix.fillText(columnChars[i][dropRow % colH], x, y);
					ctxMatrix.shadowBlur = 0;
					for (let t = 1; t < 25; t++) {
						const trailRow = (dropRow - t + colH) % colH;
						const trailY = (dropRow - t) * fontSize;
						if (trailY < 0 || trailY > H()) continue;
						const alpha = (1 - t / 25) * brightness * 0.6;
						if (alpha <= 0) continue;
						if (Math.random() > 0.995) columnChars[i][trailRow] = charArr[Math.floor(Math.random() * charArr.length)];
						const g = Math.floor(255 * (1 - t / 25));
						ctxMatrix.fillStyle = `rgba(0,${g},${Math.floor(g * 0.5)},${alpha})`;
						ctxMatrix.fillText(columnChars[i][trailRow], x, trailY);
					}
				}
				drops[i] += columnSpeeds[i];
				if (drops[i] * fontSize > H() && Math.random() > 0.975) {
					drops[i] = Math.random() * -20;
					columnSpeeds[i] = 0.3 + Math.random() * 0.8;
				}
			}
		}

		/* ── PARTICLES ── */
		const particles: Particle[] = [];
		for (let i = 0; i < 400; i++) particles.push(new Particle(W(), H()));

		function drawParticles() {
			ctxParticle.clearRect(0, 0, W(), H());
			ctxParticle.lineWidth = 0.5;
			for (let i = 0; i < particles.length; i++) {
				for (let j = i + 1; j < particles.length; j++) {
					const dx = particles[i].x - particles[j].x;
					const dy = particles[i].y - particles[j].y;
					const d2 = dx * dx + dy * dy;
					if (d2 < 8000) {
						ctxParticle.strokeStyle = `rgba(0,255,136,${(1 - d2 / 8000) * 0.15})`;
						ctxParticle.beginPath();
						ctxParticle.moveTo(particles[i].x, particles[i].y);
						ctxParticle.lineTo(particles[j].x, particles[j].y);
						ctxParticle.stroke();
					}
				}
			}
			particles.forEach(p => {
				p.update(mouseX, mouseY, W(), H());
				const pa = p.alpha + Math.sin(p.pulse) * 0.15;
				const sz = p.size * (0.8 + p.depth * 0.4);
				ctxParticle.beginPath();
				ctxParticle.arc(p.x, p.y, sz, 0, Math.PI * 2);
				ctxParticle.fillStyle = `rgba(0,255,136,${pa})`;
				ctxParticle.fill();
				if (p.depth > 0.7) {
					ctxParticle.shadowBlur = 8; ctxParticle.shadowColor = 'rgba(0,255,136,0.3)';
					ctxParticle.fill(); ctxParticle.shadowBlur = 0;
				}
			});
		}

		/* ── TRAIL ── */
		function drawTrail() {
			ctxTrail.clearRect(0, 0, W(), H());
			if (trailPoints.length < 2) return;
			trailPoints.forEach(p => { p.life -= 0.025; });
			for (let i = 1; i < trailPoints.length; i++) {
				const p = trailPoints[i]; const prev = trailPoints[i - 1];
				if (p.life <= 0) continue;
				const width = p.life * 4;
				ctxTrail.beginPath(); ctxTrail.moveTo(prev.x, prev.y); ctxTrail.lineTo(p.x, p.y);
				ctxTrail.strokeStyle = `rgba(0,255,136,${p.life * 0.4})`; ctxTrail.lineWidth = width;
				ctxTrail.lineCap = 'round'; ctxTrail.stroke();
				ctxTrail.beginPath(); ctxTrail.moveTo(prev.x, prev.y); ctxTrail.lineTo(p.x, p.y);
				ctxTrail.strokeStyle = `rgba(180,255,220,${p.life * 0.6})`; ctxTrail.lineWidth = width * 0.3;
				ctxTrail.stroke();
			}
			while (trailPoints.length && trailPoints[0].life <= 0) trailPoints.shift();
		}

		/* ── GRID ── */
		let gridTime = 0;
		function drawGrid() {
			ctxGrid.clearRect(0, 0, W(), H());
			gridTime += 0.008;
			const cx = W() / 2, cy = H() / 2, horizon = H() * 0.45, depth = 30;
			const mx = (mouseX / W() - 0.5) * 50, my = (mouseY / H() - 0.5) * 20;
			ctxGrid.strokeStyle = 'rgba(0,255,136,0.04)'; ctxGrid.lineWidth = 0.5;
			for (let i = 0; i < depth; i++) {
				const t = i / depth;
				const y = horizon + (H() - horizon) * Math.pow(t, 0.7);
				const wave = Math.sin(gridTime * 2 + i * 0.3) * 3 * (1 - t);
				const halfW = W() * 0.8 * (0.3 + t * 0.7);
				ctxGrid.globalAlpha = 0.03 + t * 0.06;
				ctxGrid.beginPath();
				ctxGrid.moveTo(cx - halfW + mx * t, y + wave + my * t);
				ctxGrid.lineTo(cx + halfW + mx * t, y + wave + my * t);
				ctxGrid.stroke();
			}
			const vLines = 25;
			for (let i = -vLines / 2; i <= vLines / 2; i++) {
				ctxGrid.globalAlpha = 0.03 * (1 - Math.abs(i) / (vLines / 2) * 0.5);
				ctxGrid.beginPath();
				ctxGrid.moveTo(cx + i * 40 * 3 + mx, H());
				ctxGrid.lineTo(cx + i * 40 * 0.3 + mx * 0.3, horizon + my * 0.3);
				ctxGrid.stroke();
			}
			ctxGrid.globalAlpha = 1;
		}

		/* ── CHART ── */
		type Candle = { open: number; close: number; high: number; low: number; volume: number; bull: boolean; alpha: number };
		const chartCandles: Candle[] = [];
		let chartPrice = 5890;
		let chartVisible = false;
		let chartMA: number[] = [];

		function addCandle() {
			const open = chartPrice;
			const vol = 3 + Math.random() * 12;
			const bias = Math.sin(chartCandles.length * 0.1) * 3;
			chartPrice += (Math.random() - 0.47) * vol + bias * 0.3;
			const close = chartPrice;
			const high = Math.max(open, close) + Math.random() * vol * 0.4;
			const low = Math.min(open, close) - Math.random() * vol * 0.4;
			chartCandles.push({ open, close, high, low, volume: Math.random() * 100 + 20, bull: close >= open, alpha: 0 });
			if (chartCandles.length > 60) chartCandles.shift();
			chartMA = chartCandles.map((_, i) => {
				const sl = chartCandles.slice(Math.max(0, i - 9), i + 1);
				return sl.reduce((s, c) => s + c.close, 0) / sl.length;
			});
			gsap.to(chartCandles[chartCandles.length - 1], { alpha: 1, duration: 0.3 });
		}

		function drawChart() {
			ctxChart.clearRect(0, 0, W(), H());
			if (!chartVisible || chartCandles.length < 2) return;
			const pad = { left: W() * 0.28, right: W() * 0.28, top: H() * 0.3, bottom: H() * 0.35 };
			const cW = W() - pad.left - pad.right, cH = H() - pad.top - pad.bottom;
			const prices = chartCandles.flatMap(c => [c.high, c.low]);
			const minP = Math.min(...prices) - 5, maxP = Math.max(...prices) + 5;
			const range = maxP - minP;
			const cw = Math.min(cW / chartCandles.length * 0.7, 12);
			const gap = cW / chartCandles.length;
			const toY = (p: number) => pad.top + cH - ((p - minP) / range) * cH;
			const toX = (i: number) => pad.left + i * gap + gap / 2;
			const maxVol = Math.max(...chartCandles.map(c => c.volume));
			chartCandles.forEach((c, i) => {
				const x = toX(i);
				ctxChart.globalAlpha = c.alpha * 0.1;
				ctxChart.fillStyle = c.bull ? '#00ff88' : '#ff3344';
				ctxChart.fillRect(x - cw / 2, pad.top + cH - (c.volume / maxVol) * cH * 0.2, cw, (c.volume / maxVol) * cH * 0.2);
			});
			if (chartMA.length > 1) {
				ctxChart.beginPath(); ctxChart.moveTo(toX(0), toY(chartMA[0]));
				for (let i = 1; i < chartMA.length; i++) ctxChart.lineTo(toX(i), toY(chartMA[i]));
				ctxChart.strokeStyle = 'rgba(0,170,255,0.3)'; ctxChart.lineWidth = 1.5;
				ctxChart.globalAlpha = 0.6; ctxChart.stroke(); ctxChart.globalAlpha = 1;
			}
			chartCandles.forEach((c, i) => {
				const x = toX(i);
				ctxChart.globalAlpha = c.alpha * 0.8;
				ctxChart.strokeStyle = c.bull ? 'rgba(0,255,136,0.5)' : 'rgba(255,51,68,0.5)';
				ctxChart.lineWidth = 1;
				ctxChart.beginPath(); ctxChart.moveTo(x, toY(c.high)); ctxChart.lineTo(x, toY(c.low)); ctxChart.stroke();
				ctxChart.fillStyle = c.bull ? '#00ff88' : '#ff3344';
				const bodyTop = Math.min(toY(c.open), toY(c.close));
				const bodyH = Math.abs(toY(c.close) - toY(c.open)) || 1;
				ctxChart.fillRect(x - cw / 2, bodyTop, cw, bodyH);
				if (i === chartCandles.length - 1) {
					ctxChart.shadowBlur = 20; ctxChart.shadowColor = c.bull ? '#00ff88' : '#ff3344';
					ctxChart.fillRect(x - cw / 2, bodyTop, cw, bodyH); ctxChart.shadowBlur = 0;
				}
			});
			const lastPrice = chartCandles[chartCandles.length - 1].close;
			const priceY = toY(lastPrice);
			ctxChart.setLineDash([4, 4]); ctxChart.strokeStyle = 'rgba(0,255,136,0.2)'; ctxChart.lineWidth = 1;
			ctxChart.globalAlpha = 0.5;
			ctxChart.beginPath(); ctxChart.moveTo(pad.left, priceY); ctxChart.lineTo(W() - pad.right, priceY); ctxChart.stroke();
			ctxChart.setLineDash([]); ctxChart.globalAlpha = 0.8;
			ctxChart.fillStyle = '#00ff88'; ctxChart.font = '10px "JetBrains Mono"';
			ctxChart.fillText(lastPrice.toFixed(2), W() - pad.right + 5, priceY + 3);
			ctxChart.globalAlpha = 1;
		}

		/* ── HUD ── */
		let hudTime = 0;
		function drawHUD() {
			ctxHud.clearRect(0, 0, W(), H());
			hudTime += 0.01;
			const cx = W() / 2, cy = H() / 2;
			const minDim = Math.min(W(), H());
			ctxHud.save(); ctxHud.translate(cx, cy); ctxHud.rotate(hudTime * 0.3);
			ctxHud.strokeStyle = 'rgba(0,255,136,0.05)'; ctxHud.lineWidth = 1;
			ctxHud.beginPath(); ctxHud.arc(0, 0, minDim * 0.38, 0, Math.PI * 2); ctxHud.stroke();
			for (let i = 0; i < 60; i++) {
				const angle = (i / 60) * Math.PI * 2;
				const r1 = minDim * 0.36; const r2 = minDim * (i % 5 === 0 ? 0.34 : 0.355);
				ctxHud.strokeStyle = `rgba(0,255,136,${i % 5 === 0 ? 0.1 : 0.04})`;
				ctxHud.beginPath();
				ctxHud.moveTo(Math.cos(angle) * r1, Math.sin(angle) * r1);
				ctxHud.lineTo(Math.cos(angle) * r2, Math.sin(angle) * r2);
				ctxHud.stroke();
			}
			ctxHud.restore();
			ctxHud.save(); ctxHud.translate(cx, cy); ctxHud.rotate(-hudTime * 0.5);
			ctxHud.strokeStyle = 'rgba(0,255,136,0.03)'; ctxHud.setLineDash([20, 10]);
			ctxHud.beginPath(); ctxHud.arc(0, 0, minDim * 0.3, 0, Math.PI * 2); ctxHud.stroke();
			ctxHud.setLineDash([]); ctxHud.restore();
			const pulseR = ((hudTime * 60) % (minDim * 0.4));
			ctxHud.strokeStyle = `rgba(0,255,136,${(1 - pulseR / (minDim * 0.4)) * 0.06})`;
			ctxHud.lineWidth = 1;
			ctxHud.beginPath(); ctxHud.arc(cx, cy, pulseR, 0, Math.PI * 2); ctxHud.stroke();
			// Crosshair
			ctxHud.strokeStyle = 'rgba(0,255,136,0.15)'; ctxHud.lineWidth = 0.5; ctxHud.setLineDash([3, 6]);
			[[mouseX - 40, mouseY, mouseX - 10, mouseY],[mouseX + 10, mouseY, mouseX + 40, mouseY],
			 [mouseX, mouseY - 40, mouseX, mouseY - 10],[mouseX, mouseY + 10, mouseX, mouseY + 40]].forEach(([x1,y1,x2,y2]) => {
				ctxHud.beginPath(); ctxHud.moveTo(x1, y1); ctxHud.lineTo(x2, y2); ctxHud.stroke();
			});
			ctxHud.setLineDash([]);
			ctxHud.fillStyle = 'rgba(0,255,136,0.15)'; ctxHud.font = '9px "JetBrains Mono"';
			ctxHud.fillText(`${mouseX.toFixed(0)}, ${mouseY.toFixed(0)}`, mouseX + 15, mouseY - 15);
		}

		/* ── DATA PANELS ── */
		function generateOrderBook() {
			const mid = 5892.50;
			let html = '';
			for (let i = 5; i >= 1; i--) {
				const price = (mid + i * 0.25).toFixed(2);
				const size = Math.floor(Math.random() * 200 + 10);
				html += `<div class="mt-ob-row mt-ask"><div class="mt-ob-bg" style="width:${Math.min(size / 200 * 100, 100)}%"></div><span class="mt-ob-price">${price}</span><span class="mt-ob-size">${size}</span></div>`;
			}
			html += `<div style="height:1px;background:rgba(0,255,136,0.1);margin:4px 0"></div>`;
			for (let i = 0; i < 5; i++) {
				const price = (mid - i * 0.25).toFixed(2);
				const size = Math.floor(Math.random() * 200 + 10);
				html += `<div class="mt-ob-row mt-bid"><div class="mt-ob-bg" style="width:${Math.min(size / 200 * 100, 100)}%"></div><span class="mt-ob-price">${price}</span><span class="mt-ob-size">${size}</span></div>`;
			}
			orderbookContent.innerHTML = html;
		}

		function generatePositions() {
			const pos = [
				{ sym: 'ES', side: 'LONG', qty: 5, entry: '5,878.25', pnl: '+$4,250', up: true },
				{ sym: 'NQ', side: 'LONG', qty: 2, entry: '21,190.50', pnl: '+$6,120', up: true },
				{ sym: 'SPX 0DTE', side: 'CALL', qty: 10, entry: '$12.40', pnl: '+$8,900', up: true },
				{ sym: 'VIX', side: 'SHORT', qty: 3, entry: '19.85', pnl: '+$1,340', up: true },
			];
			positionsContent.innerHTML = pos.map(p =>
				`<div class="mt-row"><span>${p.sym} ${p.side} x${p.qty}</span><span class="mt-val ${p.up ? 'mt-up' : 'mt-down'}">${p.pnl}</span></div>`
			).join('') + `<div style="margin-top:10px;padding-top:8px;border-top:1px solid rgba(0,255,136,0.1)"><div class="mt-row"><span style="color:#00ff88">TOTAL P&L</span><span class="mt-val mt-up" style="font-weight:700">+$20,610</span></div></div>`;
		}

		function generateTicker() {
			const items = [
				{ sym: 'SPX', price: '5,892.47', chg: '+2.31%', up: true },
				{ sym: 'NQ', price: '21,340.12', chg: '+1.87%', up: true },
				{ sym: 'ES', price: '5,901.25', chg: '+0.94%', up: true },
				{ sym: 'VIX', price: '18.42', chg: '-12.4%', up: false },
				{ sym: 'AAPL', price: '234.67', chg: '+3.2%', up: true },
				{ sym: 'TSLA', price: '412.89', chg: '+5.1%', up: true },
				{ sym: 'NVDA', price: '892.34', chg: '+2.8%', up: true },
				{ sym: 'AMZN', price: '198.45', chg: '-0.4%', up: false },
				{ sym: 'META', price: '567.12', chg: '+1.9%', up: true },
				{ sym: 'BTC', price: '97,234', chg: '+4.2%', up: true },
				{ sym: 'ETH', price: '3,891', chg: '+3.8%', up: true },
			];
			tickerInner.innerHTML = [0, 1].flatMap(() => items.map(it =>
				`<span class="mt-ticker-item"><span class="mt-sym">${it.sym}</span><span class="mt-price">${it.price}</span><span class="mt-chg ${it.up ? 'mt-up' : 'mt-down'}">${it.chg}</span></span>`
			)).join('');
		}

		/* ── GLITCH ── */
		function triggerGlitch() {
			const y1 = Math.random() * H(), y2 = Math.random() * H();
			gsap.set(glitchSlice1, { top: y1, width: '100%' });
			gsap.set(glitchSlice2, { top: y2, width: '60%', left: Math.random() * 40 + '%' });
			const tl = gsap.timeline();
			tl.to(glitchSlice1, { opacity: 0.3, duration: 0.03 })
				.to(glitchSlice2, { opacity: 0.2, duration: 0.03 }, '<')
				.set(glitchSlice1, { opacity: 0 }, '+=0.04')
				.set(glitchSlice2, { opacity: 0 });
			gsap.to(centerTitle, {
				x: (Math.random() - 0.5) * 15, skewX: (Math.random() - 0.5) * 3, duration: 0.05,
				onComplete: () => { gsap.to(centerTitle, { x: 0, skewX: 0, duration: 0.1 }); }
			});
			gsap.delayedCall(Math.random() * 4 + 2, triggerGlitch);
		}

		function triggerAlert() {
			const color = Math.random() > 0.3 ? '#00ff88' : '#ff3344';
			gsap.to(alertFlash, {
				borderColor: color, opacity: 0.4, duration: 0.05,
				onComplete: () => { gsap.to(alertFlash, { opacity: 0, duration: 0.3 }); }
			});
			gsap.delayedCall(Math.random() * 6 + 3, triggerAlert);
		}

		/* ── CLOCK ── */
		function updateClock() {
			const now = new Date();
			const ms = String(now.getMilliseconds()).padStart(3, '0');
			if (clockEl) clockEl.textContent = now.toTimeString().split(' ')[0] + '.' + ms;
			rafIds.push(requestAnimationFrame(updateClock));
		}

		/* ── FPS ── */
		function updateFPS() {
			frameCount++;
			const now = performance.now();
			if (now - lastTime >= 500) {
				const fps = Math.round(frameCount / ((now - lastTime) / 1000));
				if (fpsEl) fpsEl.textContent = 'FPS: ' + fps;
				frameCount = 0; lastTime = now;
			}
		}

		/* ── SCANLINE ── */
		gsap.to(scanlineEl, { top: '100%', duration: 4, ease: 'none', repeat: -1 });

		/* ── MAIN LOOP ── */
		function mainLoop() {
			if (systemOnline) {
				drawMatrix(); drawParticles(); drawTrail(); drawGrid(); drawChart(); drawHUD(); updateFPS();
			}
			rafIds.push(requestAnimationFrame(mainLoop));
		}
		mainLoop();

		/* ── BOOT SEQUENCE ── */
		const bootLines = bootScreen.querySelectorAll<HTMLElement>('.mt-boot-line');
		const bootTl = gsap.timeline();

		gsap.to(bootScreen.querySelector('.mt-boot-cursor'), { opacity: 0, duration: 0.4, repeat: -1, yoyo: true, ease: 'steps(1)' });

		bootLines.forEach(line => {
			const delay = parseInt(line.dataset.delay ?? '0') / 1000;
			bootTl.to(line, {
				opacity: 1, duration: 0.02,
				onStart: () => {
					if (line.classList.contains('mt-error')) gsap.to(bootScreen, { x: 5, duration: 0.03, yoyo: true, repeat: 3 });
				}
			}, delay);
		});

		bootTl.to(bootScreen.querySelector('.mt-boot-progress'), { opacity: 1, duration: 0.3 }, 2.2);
		bootTl.to(bootScreen.querySelector('.mt-boot-progress-fill'), { width: '100%', duration: 1.5, ease: 'power2.inOut' }, 2.2);

		bootTl.add(() => {
			systemOnline = true;
			generateOrderBook(); generatePositions(); generateTicker(); updateClock();
			gsap.delayedCall(3, triggerGlitch);
			gsap.delayedCall(5, triggerAlert);
			setInterval(generateOrderBook, 1500);
			for (let i = 0; i < 5; i++) addCandle();
			setInterval(addCandle, 800);
		}, 4);

		/* ── REVEAL ── */
		const revealTl = gsap.timeline({ delay: 4.2 });
		revealTl.to(bootScreen, { background: '#001a0d', duration: 0.1 });
		revealTl.to(bootScreen, { background: '#000', duration: 0.1 });
		revealTl.to(bootScreen, {
			opacity: 0, duration: 0.8, ease: 'power2.in',
			onComplete: () => { bootScreen.style.display = 'none'; }
		}, 0.3);
		revealTl.to(topBar, { opacity: 1, duration: 0.5 }, 0.8);
		revealTl.to(section.querySelectorAll('.mt-bracket'), { opacity: 1, duration: 0.3, stagger: 0.05 }, 1);

		// Flash
		revealTl.to(alertFlash, { borderColor: '#00ff88', opacity: 0.6, duration: 0.03 }, 1.5);
		revealTl.to(alertFlash, { opacity: 0, duration: 0.3 }, 1.55);

		// Title slam
		revealTl.to(centerTitle, { opacity: 1, filter: 'blur(0px)', scale: 1, duration: 0.5, ease: 'power4.out' }, 1.5);

		// Screen shake
		revealTl.add(() => {
			const el = section.querySelector<HTMLElement>('.mt-center-content')!;
			for (let i = 0; i < 15; i++) {
				const d = 1 - i / 15;
				gsap.set(el, { x: (Math.random() - 0.5) * 30 * d, y: (Math.random() - 0.5) * 20 * d, delay: i * 0.025 });
			}
			gsap.set(el, { x: 0, y: 0, delay: 15 * 0.025 });
		}, 1.5);

		// Subtitle char-by-char
		const subText = 'REVOLUTION TRADING PROS';
		centerSub.innerHTML = subText.split('').map(c =>
			c === ' ' ? ' ' : `<span class="mt-char">${c}</span>`
		).join('');
		revealTl.to(section.querySelectorAll('.mt-char'), {
			opacity: 1, y: 0, duration: 0.03, stagger: 0.03, ease: 'none'
		}, 2.2);

		revealTl.to(centerTagline, { opacity: 1, duration: 0.8 }, 3);
		revealTl.add(() => { chartVisible = true; }, 2);

		gsap.set(panelOrderbook, { x: -40 });
		revealTl.to(panelOrderbook, { opacity: 1, x: 0, duration: 0.8, ease: 'power3.out' }, 2.5);
		gsap.set(panelPositions, { x: 40 });
		revealTl.to(panelPositions, { opacity: 1, x: 0, duration: 0.8, ease: 'power3.out' }, 2.7);
		revealTl.to(tickerTape, { opacity: 1, duration: 0.5 }, 3);

		// Ambient glow
		gsap.to(centerTitle, {
			textShadow: '0 0 80px rgba(0,255,136,0.3), 0 0 120px rgba(0,255,136,0.1)',
			duration: 2, repeat: -1, yoyo: true, ease: 'sine.inOut'
		});

		return () => {
			rafIds.forEach(id => cancelAnimationFrame(id));
			window.removeEventListener('resize', resizeAll);
			window.removeEventListener('resize', initMatrix);
		};
	});
</script>

<section class="mt-section" bind:this={section}>
	<!-- Canvas layers -->
	<canvas class="mt-canvas" style="z-index:1" bind:this={matrixCanvas}></canvas>
	<canvas class="mt-canvas" style="z-index:4" bind:this={trailCanvas}></canvas>
	<canvas class="mt-canvas" style="z-index:2" bind:this={particleCanvas}></canvas>
	<canvas class="mt-canvas" style="z-index:3" bind:this={gridCanvas}></canvas>
	<canvas class="mt-canvas" style="z-index:5" bind:this={chartCanvas}></canvas>
	<canvas class="mt-canvas" style="z-index:8" bind:this={hudCanvas}></canvas>

	<!-- Custom cursor -->
	<div class="mt-cursor" bind:this={cursorEl}></div>
	<div class="mt-cursor-dot" bind:this={cursorDot}></div>

	<!-- Boot screen -->
	<div class="mt-boot-screen" bind:this={bootScreen}>
		<div class="mt-boot-line mt-dim" data-delay="0">[BIOS] Initializing quantum trading core...</div>
		<div class="mt-boot-line" data-delay="200">████████ MEMORY CHECK: 128TB OK</div>
		<div class="mt-boot-line mt-dim" data-delay="400">[SYS] Loading neural pattern recognition v9.7.2...</div>
		<div class="mt-boot-line" data-delay="600">[NET] Connecting to dark fiber network... <span style="color:#00ffcc">ESTABLISHED</span></div>
		<div class="mt-boot-line mt-dim" data-delay="800">[MKT] Subscribing to L3 market data feeds...</div>
		<div class="mt-boot-line" data-delay="1000">[MKT] CME ✓ NYSE ✓ NASDAQ ✓ CBOE ✓ ICE ✓</div>
		<div class="mt-boot-line mt-error" data-delay="1200">⚠ WARNING: Volatility index exceeds threshold</div>
		<div class="mt-boot-line mt-dim" data-delay="1400">[ALGO] Calibrating execution engine...</div>
		<div class="mt-boot-line" data-delay="1600">[ALGO] Latency: 0.003ms — OPTIMAL</div>
		<div class="mt-boot-line mt-highlight" data-delay="1800">[RTP] Revolution Trading Protocol: ARMED</div>
		<div class="mt-boot-line mt-highlight" data-delay="2000">[RTP] Explosive Swings Module: LOADED</div>
		<div class="mt-boot-line" data-delay="2200" style="margin-top:10px">SYSTEM READY.<span class="mt-boot-cursor"></span></div>
		<div class="mt-boot-progress"><div class="mt-boot-progress-fill"></div></div>
	</div>

	<!-- Post effects -->
	<div class="mt-noise"></div>
	<div class="mt-vignette"></div>
	<div class="mt-crt"></div>
	<div class="mt-scanline" bind:this={scanlineEl}></div>
	<div class="mt-alert-flash" bind:this={alertFlash}></div>
	<div class="mt-glitch-slice" bind:this={glitchSlice1}></div>
	<div class="mt-glitch-slice" style="height:2px" bind:this={glitchSlice2}></div>

	<!-- HUD -->
	<div class="mt-top-bar" bind:this={topBar}>
		<div class="mt-logo">◈ RTP TERMINAL v4.0</div>
		<div class="mt-status">
			<span class="mt-live">LIVE</span>
			<span bind:this={clockEl}>00:00:00.000</span>
			<span>LAT: 0.003ms</span>
			<span bind:this={fpsEl}>FPS: 60</span>
		</div>
	</div>

	<div class="mt-bracket mt-tl"></div>
	<div class="mt-bracket mt-tr"></div>
	<div class="mt-bracket mt-bl"></div>
	<div class="mt-bracket mt-br"></div>

	<!-- Center -->
	<div class="mt-center-content">
		<h1 class="mt-center-title" bind:this={centerTitle}>RTP</h1>
		<div class="mt-center-sub" bind:this={centerSub}>REVOLUTION TRADING PROS</div>
		<div class="mt-center-tagline" bind:this={centerTagline}>[ THE MOVE PRIOR TO THE MOVE ]</div>
	</div>

	<!-- Data panels -->
	<div class="mt-data-panel mt-panel-orderbook" bind:this={panelOrderbook}>
		<div class="mt-panel-header">◈ ORDER BOOK — ES</div>
		<div bind:this={orderbookContent}></div>
	</div>
	<div class="mt-data-panel mt-panel-positions" bind:this={panelPositions}>
		<div class="mt-panel-header">◈ LIVE POSITIONS</div>
		<div bind:this={positionsContent}></div>
	</div>

	<!-- Ticker -->
	<div class="mt-ticker-tape" bind:this={tickerTape}>
		<div class="mt-ticker-inner" bind:this={tickerInner}></div>
	</div>
</section>

<style>
	.mt-section {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #000;
		color: #00ff88;
		font-family: 'JetBrains Mono', monospace;
		cursor: none;
	}

	.mt-canvas {
		position: absolute;
		inset: 0;
		width: 100%; height: 100%;
		pointer-events: none;
	}

	.mt-cursor {
		position: absolute;
		width: 20px; height: 20px;
		border: 1px solid #00ff88;
		border-radius: 50%;
		pointer-events: none;
		z-index: 99999;
		mix-blend-mode: difference;
		transition: width 0.2s, height 0.2s;
	}
	.mt-cursor-dot {
		position: absolute;
		width: 4px; height: 4px;
		background: #00ff88;
		border-radius: 50%;
		pointer-events: none;
		z-index: 99999;
	}

	.mt-boot-screen {
		position: absolute;
		inset: 0;
		z-index: 100;
		background: #000;
		display: flex;
		flex-direction: column;
		justify-content: center;
		padding: 10% 15%;
		overflow: hidden;
	}
	.mt-boot-line {
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.65rem;
		color: #00ff88;
		opacity: 0;
		line-height: 1.8;
		letter-spacing: 0.05em;
	}
	.mt-dim { color: rgba(0,255,136,0.3); }
	.mt-error { color: #ff3344; }
	.mt-highlight { color: #00ffcc; font-weight: 700; }
	.mt-boot-cursor {
		display: inline-block;
		width: 8px; height: 13px;
		background: #00ff88;
		vertical-align: middle;
		margin-left: 2px;
	}
	.mt-boot-progress {
		width: 300px; height: 2px;
		background: rgba(0,255,136,0.1);
		margin-top: 20px;
		position: relative;
		opacity: 0;
	}
	.mt-boot-progress-fill {
		position: absolute;
		top: 0; left: 0;
		height: 100%; width: 0;
		background: #00ff88;
		box-shadow: 0 0 10px #00ff88;
	}

	.mt-noise {
		position: absolute; inset: 0; z-index: 50;
		pointer-events: none; opacity: 0.03;
		background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 512 512' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
		background-size: 256px;
	}
	.mt-vignette {
		position: absolute; inset: 0; z-index: 45; pointer-events: none;
		background: radial-gradient(ellipse at center, transparent 50%, rgba(0,0,0,0.6) 100%);
	}
	.mt-crt {
		position: absolute; inset: 0; z-index: 48; pointer-events: none;
		background: radial-gradient(ellipse at center, transparent 60%, rgba(0,20,10,0.15) 100%);
	}
	.mt-crt::before {
		content: '';
		position: absolute; inset: 0;
		background: repeating-linear-gradient(0deg, transparent, transparent 1px, rgba(0,0,0,0.03) 1px, rgba(0,0,0,0.03) 2px);
	}
	.mt-scanline {
		position: absolute; left: 0; right: 0; height: 3px; z-index: 25;
		background: linear-gradient(90deg, transparent, rgba(0,255,136,0.06), transparent);
		pointer-events: none;
	}
	.mt-alert-flash {
		position: absolute; inset: 0; z-index: 30; pointer-events: none;
		border: 2px solid transparent; opacity: 0;
	}
	.mt-glitch-slice {
		position: absolute; left: 0; right: 0; height: 4px; z-index: 28;
		background: #00ff88; opacity: 0; pointer-events: none;
		mix-blend-mode: overlay;
	}

	/* TOP BAR */
	.mt-top-bar {
		position: absolute; top: 0; left: 0; right: 0; height: 40px; z-index: 20;
		display: flex; align-items: center; justify-content: space-between;
		padding: 0 30px;
		border-bottom: 1px solid rgba(0,255,136,0.08);
		background: rgba(0,0,0,0.6); backdrop-filter: blur(10px);
		opacity: 0;
	}
	.mt-logo { font-family: 'Orbitron', sans-serif; font-size: 0.65rem; letter-spacing: 0.4em; color: #00ff88; }
	.mt-status { display: flex; gap: 25px; font-size: 0.55rem; letter-spacing: 0.15em; color: rgba(0,255,136,0.4); }
	.mt-live { color: #00ff88; }
	.mt-live::before {
		content: ''; display: inline-block; width: 5px; height: 5px;
		border-radius: 50%; background: #00ff88; margin-right: 6px;
		vertical-align: middle; box-shadow: 0 0 8px #00ff88;
	}

	/* BRACKETS */
	.mt-bracket { position: absolute; z-index: 15; opacity: 0; }
	.mt-bracket::before, .mt-bracket::after { content: ''; position: absolute; background: #00ff88; }
	.mt-tl { top: 50px; left: 20px; }
	.mt-tr { top: 50px; right: 20px; }
	.mt-bl { bottom: 20px; left: 20px; }
	.mt-br { bottom: 20px; right: 20px; }
	.mt-tl::before, .mt-bl::before { left: 0; width: 40px; height: 1px; }
	.mt-tr::before, .mt-br::before { right: 0; width: 40px; height: 1px; }
	.mt-tl::after, .mt-tr::after { top: 0; height: 40px; width: 1px; }
	.mt-bl::after, .mt-br::after { bottom: 0; height: 40px; width: 1px; }
	.mt-tl::before { top: 0; } .mt-tl::after { left: 0; }
	.mt-tr::before { top: 0; } .mt-tr::after { right: 0; }
	.mt-bl::before { bottom: 0; } .mt-bl::after { left: 0; }
	.mt-br::before { bottom: 0; } .mt-br::after { right: 0; }

	/* CENTER */
	.mt-center-content {
		position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
		z-index: 12; text-align: center; pointer-events: none;
	}
	.mt-center-title {
		font-family: 'Anton', sans-serif;
		font-size: clamp(5rem, 18vw, 16rem);
		line-height: 0.85; color: transparent;
		background: linear-gradient(180deg, #00ff88 0%, #00aa55 40%, #004422 100%);
		-webkit-background-clip: text; background-clip: text;
		opacity: 0; filter: blur(30px); transform: scale(3);
	}
	.mt-center-sub {
		font-family: 'Orbitron', sans-serif;
		font-size: clamp(0.6rem, 1.5vw, 0.9rem);
		letter-spacing: 0.8em; color: #00ff88;
		margin-top: 20px; opacity: 0;
	}
	:global(.mt-char) { display: inline-block; opacity: 0; transform: translateY(20px); }
	.mt-center-tagline {
		font-family: 'JetBrains Mono', monospace; font-size: 0.6rem;
		letter-spacing: 0.3em; color: rgba(0,255,136,0.3);
		margin-top: 40px; opacity: 0;
	}

	/* DATA PANELS */
	.mt-data-panel {
		position: absolute; z-index: 11;
		background: rgba(0,10,5,0.5);
		border: 1px solid rgba(0,255,136,0.08);
		backdrop-filter: blur(5px);
		padding: 15px; opacity: 0;
		font-size: 0.55rem; letter-spacing: 0.1em;
	}
	.mt-panel-header {
		color: #00ff88; font-size: 0.5rem; letter-spacing: 0.3em;
		margin-bottom: 10px; padding-bottom: 6px;
		border-bottom: 1px solid rgba(0,255,136,0.1);
		text-transform: uppercase;
	}
	.mt-panel-orderbook { top: 80px; left: 20px; width: 200px; }
	.mt-panel-positions { top: 80px; right: 20px; width: 220px; }
	:global(.mt-ob-row) { display: flex; justify-content: space-between; position: relative; padding: 2px 4px; font-size: 0.5rem; }
	:global(.mt-ob-bg) { position: absolute; top: 0; bottom: 0; right: 0; opacity: 0.08; }
	:global(.mt-bid .mt-ob-bg) { background: #00ff88; }
	:global(.mt-ask .mt-ob-bg) { background: #ff3344; }
	:global(.mt-bid .mt-ob-price) { color: rgba(0,255,136,0.6); }
	:global(.mt-ask .mt-ob-price) { color: rgba(255,51,68,0.6); }
	:global(.mt-ob-size) { color: rgba(0,255,136,0.25); }
	:global(.mt-row) { display: flex; justify-content: space-between; gap: 20px; padding: 3px 0; color: rgba(0,255,136,0.4); }
	:global(.mt-val) { color: rgba(0,255,136,0.7); }
	:global(.mt-up) { color: #00ff88 !important; }
	:global(.mt-down) { color: #ff3344 !important; }

	/* TICKER */
	.mt-ticker-tape {
		position: absolute; bottom: 0; left: 0; right: 0; height: 30px; z-index: 20;
		display: flex; align-items: center; overflow: hidden;
		background: rgba(0,0,0,0.7); border-top: 1px solid rgba(0,255,136,0.08);
		opacity: 0;
	}
	.mt-ticker-inner {
		display: flex; gap: 40px; white-space: nowrap; font-size: 0.55rem;
		letter-spacing: 0.1em;
		animation: mtTickerScroll 30s linear infinite;
	}
	@keyframes mtTickerScroll { 0% { transform: translateX(0); } 100% { transform: translateX(-50%); } }
	:global(.mt-ticker-item .mt-sym) { color: rgba(0,255,136,0.5); margin-right: 8px; }
	:global(.mt-ticker-item .mt-price) { color: rgba(255,255,255,0.6); margin-right: 8px; }
	:global(.mt-ticker-item .mt-chg.mt-up) { color: #00ff88; }
	:global(.mt-ticker-item .mt-chg.mt-down) { color: #ff3344; }
</style>
