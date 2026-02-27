<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let s7a: HTMLCanvasElement, s7b: HTMLCanvasElement;
	let s8a: HTMLCanvasElement, s8b: HTMLCanvasElement;
	let s9a: HTMLCanvasElement, s9b: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }
	function shake(el: HTMLElement, intensity = 20, dur = 0.5) { const s = Math.floor(dur / 0.016); for (let i = 0; i < s; i++) { const f = Math.pow(1 - i / s, 2); gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.6, delay: i * 0.016 }); } gsap.set(el, { x: 0, y: 0, delay: s * 0.016 }); }

	function initThePit() {
		const cv = [s7a, s7b];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s7a.width = s7a.parentElement!.offsetWidth, h = s7a.height = s7a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const ORDERS = ['ES BUY 50', 'NQ SELL 20', 'SPX CALL 100', 'ES BUY 200', 'FILL 5892.50', 'NQ BUY 75', 'SELL SELL SELL', 'BUY THE DIP', 'FILLED @ MKT', 'ES 5893 BID'];
		const tickets: { x: number; y: number; vx: number; vy: number; text: string; size: number; alpha: number; rot: number; color: string }[] = [];
		for (let i = 0; i < 200; i++) { const a = Math.random() * Math.PI * 2, spd = Math.random() * 3 + 0.5; tickets.push({ x: Math.random() * 2000, y: Math.random() * 1200, vx: Math.cos(a) * spd, vy: Math.sin(a) * spd, text: ORDERS[Math.floor(Math.random() * ORDERS.length)], size: 6 + Math.random() * 6, alpha: Math.random() * 0.25 + 0.05, rot: (Math.random() - 0.5) * 0.4, color: Math.random() > 0.5 ? '0,255,136' : '255,51,68' }); }
		const wave = new Array(150).fill(0);
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s7a.width; h = s7a.height; }
		window.addEventListener('resize', resize);
		function draw() { if (!on) { rafIds.push(requestAnimationFrame(draw)); return; } t += 0.008; const [xa, xb] = ctx; xa.fillStyle = 'rgba(0,0,0,0.04)'; xa.fillRect(0, 0, w, h); const mc = Math.floor(mx / 30), mr = Math.floor(my / 30); for (let dr = -3; dr <= 3; dr++) for (let dc = -3; dc <= 3; dc++) { const d = Math.sqrt(dr * dr + dc * dc); if (d > 3) continue; xa.fillStyle = `rgba(255,${Math.floor(150 + d * 20)},0,${(1 - d / 3) * 0.03})`; xa.fillRect((mc + dc) * 30, (mr + dr) * 30, 30, 30); } xb.clearRect(0, 0, w, h); tickets.forEach(tk => { const dx = mx - tk.x, dy = my - tk.y, dist = Math.sqrt(dx * dx + dy * dy); if (dist < 150) { tk.vx -= (dx / dist) * 1.5; tk.vy -= (dy / dist) * 1.5; } tk.vx += (Math.random() - 0.5) * 0.15; tk.vy += (Math.random() - 0.5) * 0.15; tk.vx *= 0.99; tk.vy *= 0.99; tk.x += tk.vx; tk.y += tk.vy; if (tk.x < -100) tk.x = w + 100; if (tk.x > w + 100) tk.x = -100; if (tk.y < -50) tk.y = h + 50; if (tk.y > h + 50) tk.y = -50; xb.save(); xb.translate(tk.x, tk.y); xb.rotate(tk.rot + Math.sin(t + tk.x * 0.01) * 0.05); xb.font = `${tk.size}px "JetBrains Mono"`; xb.fillStyle = `rgba(${tk.color},${tk.alpha})`; xb.fillText(tk.text, 0, 0); xb.restore(); }); wave.shift(); wave.push(Math.sin(t * 15) * 25 + Math.sin(t * 23) * 15 + (Math.random() - 0.5) * 40); xa.beginPath(); wave.forEach((v, i) => { const px = i / wave.length * w; i === 0 ? xa.moveTo(px, h * 0.88 + v) : xa.lineTo(px, h * 0.88 + v); }); xa.strokeStyle = 'rgba(255,200,0,0.12)'; xa.lineWidth = 1.5; xa.shadowBlur = 8; xa.shadowColor = 'rgba(255,200,0,0.2)'; xa.stroke(); xa.shadowBlur = 0; rafIds.push(requestAnimationFrame(draw)); }
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s7', start: 'top 60%', onEnter: () => { on = true; const tl = gsap.timeline({ delay: 0.5 }); tl.to('#s7f', { opacity: 0.3, duration: 0.05 }, 0.5); tl.to('#s7f', { opacity: 0, duration: 0.8 }, 0.55); tl.to('#s7t', { opacity: 1, duration: 0.5 }, 1); tl.add(() => shake(document.getElementById('s7-content')!, 15, 0.3), 1); tl.to('#s7u', { opacity: 1, duration: 0.8 }, 1.5); tl.to('#s7l', { opacity: 0.12, duration: 0.5 }, 1); } }); triggers.push(trigger);
	}

	function initVolumeProfile() {
		const cv = [s8a, s8b];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s8a.width = s8a.parentElement!.offsetWidth, h = s8a.height = s8a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const profileBars: { volume: number; isPOC: boolean; inVA: boolean }[] = [];
		const ROWS = 50; const poc = 25;
		for (let i = 0; i < ROWS; i++) { const distFromPOC = Math.abs(i - poc); const vol = Math.exp(-distFromPOC * distFromPOC / 80) * 100 + Math.random() * 15; profileBars.push({ volume: vol, isPOC: i === poc, inVA: distFromPOC < 10 }); }
		const maxVol = Math.max(...profileBars.map(b => b.volume));
		const buildProgress = { v: 0 };
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s8a.width; h = s8a.height; }
		window.addEventListener('resize', resize);
		function draw() { if (!on) { rafIds.push(requestAnimationFrame(draw)); return; } t += 0.008; const [xa, xb] = ctx; xa.fillStyle = 'rgba(0,0,0,0.1)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); const prog = buildProgress.v; const barH = h / ROWS; const startX = w * 0.15; profileBars.forEach((bar, i) => { const y = i * barH; const barW = (bar.volume / maxVol) * w * 0.5 * prog; if (bar.inVA) { xa.fillStyle = 'rgba(180,100,255,0.03)'; xa.fillRect(startX, y, w * 0.5, barH); } const alpha = bar.isPOC ? 0.8 : 0.3 + bar.volume / maxVol * 0.3; xb.fillStyle = bar.isPOC ? `rgba(255,200,50,${alpha * prog})` : `rgba(180,100,255,${alpha * prog})`; xb.fillRect(startX, y + 1, barW, barH - 2); if (bar.isPOC && prog > 0.5) { xb.shadowBlur = 15; xb.shadowColor = 'rgba(255,200,50,0.4)'; xb.fillRect(startX, y + 1, barW, barH - 2); xb.shadowBlur = 0; xb.fillStyle = `rgba(255,200,50,${prog * 0.6})`; xb.font = 'bold 9px "JetBrains Mono"'; xb.fillText('◄ POC', startX + barW + 8, y + barH * 0.7); } if (i % 5 === 0) { const price = 5850 + i * 2; xb.fillStyle = 'rgba(180,100,255,0.15)'; xb.font = '8px "JetBrains Mono"'; xb.textAlign = 'right'; xb.fillText(price.toFixed(2), startX - 8, y + barH * 0.7); xb.textAlign = 'start'; } }); if (prog > 0.5) { const vaTop = 15 * barH, vaBot = 35 * barH; xb.setLineDash([3, 3]); xb.strokeStyle = 'rgba(180,100,255,0.15)'; xb.beginPath(); xb.moveTo(startX, vaTop); xb.lineTo(w * 0.7, vaTop); xb.stroke(); xb.beginPath(); xb.moveTo(startX, vaBot); xb.lineTo(w * 0.7, vaBot); xb.stroke(); xb.setLineDash([]); xb.fillStyle = 'rgba(180,100,255,0.2)'; xb.font = '8px Orbitron'; xb.fillText('VALUE AREA HIGH', w * 0.72, vaTop + 4); xb.fillText('VALUE AREA LOW', w * 0.72, vaBot + 4); } rafIds.push(requestAnimationFrame(draw)); }
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s8', start: 'top 60%', onEnter: () => { on = true; gsap.to(buildProgress, { v: 1, duration: 3, ease: 'power2.out' }); const tl = gsap.timeline({ delay: 2.5 }); tl.to('#s8f', { opacity: 0.2, duration: 0.05 }, 0); tl.to('#s8f', { opacity: 0, duration: 1 }, 0.05); tl.to('#s8t', { opacity: 1, duration: 0.5 }, 0.3); tl.to('#s8u', { opacity: 1, duration: 0.8 }, 0.8); tl.to('#s8l', { opacity: 0.12, duration: 0.5 }, 0.5); } }); triggers.push(trigger);
	}

	function initOptionsFlow() {
		const cv = [s9a, s9b];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s9a.width = s9a.parentElement!.offsetWidth, h = s9a.height = s9a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const flows: { ticker: string; strike: number; premium: number; size: number; isCall: boolean; isSweep: boolean; exp: string; y: number; speed: number; alpha: number }[] = [];
		const TICKERS = ['SPX', 'AAPL', 'TSLA', 'NVDA', 'AMZN', 'META', 'MSFT', 'GOOGL', 'AMD', 'QQQ'];
		function addFlow() { if (!on) return; const isCall = Math.random() > 0.35; const isSweep = Math.random() > 0.7; const ticker = TICKERS[Math.floor(Math.random() * TICKERS.length)]; const strike = Math.floor(Math.random() * 50 + 150); const premium = Math.floor(Math.random() * 2000 + 50); const size = Math.floor(Math.random() * 500 + 10); const exp = `${Math.floor(Math.random() * 12 + 1)}/${Math.floor(Math.random() * 28 + 1)}`; flows.push({ ticker, strike, premium, size, isCall, isSweep, exp, y: -35, speed: 2 + Math.random() * 3, alpha: 0 }); timeouts.push(window.setTimeout(addFlow, 300 + Math.random() * 500)); }
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s9a.width; h = s9a.height; }
		window.addEventListener('resize', resize);
		function draw() { if (!on) { rafIds.push(requestAnimationFrame(draw)); return; } t += 0.008; const [xa, xb] = ctx; xa.fillStyle = 'rgba(0,0,0,0.08)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xb.fillStyle = 'rgba(255,255,255,0.1)'; xb.font = '8px Orbitron'; const headers = ['TICKER', 'STRIKE', 'EXP', 'C/P', 'SIZE', 'PREMIUM', 'TYPE']; headers.forEach((h2, i) => { xb.fillText(h2, w * 0.08 + i * w * 0.12, 20); }); xb.strokeStyle = 'rgba(255,255,255,0.05)'; xb.beginPath(); xb.moveTo(w * 0.05, 28); xb.lineTo(w * 0.95, 28); xb.stroke(); flows.forEach(f => { f.y += f.speed; f.alpha = Math.min(1, f.alpha + 0.06); const y = f.y + 35; const fade = f.alpha * (1 - Math.max(0, (f.y - h * 0.7) / (h * 0.3))); const clr = f.isCall ? '0,255,136' : '255,0,200'; const baseX = w * 0.08; if (f.isSweep) { xa.fillStyle = `rgba(${clr},${fade * 0.04})`; xa.fillRect(w * 0.05, y - 10, w * 0.9, 18); } xb.font = `${f.isSweep ? 'bold ' : ''}10px "JetBrains Mono"`; xb.fillStyle = `rgba(255,255,255,${fade * 0.5})`; xb.fillText(f.ticker, baseX, y); xb.fillText(`$${f.strike}`, baseX + w * 0.12, y); xb.fillText(f.exp, baseX + w * 0.24, y); xb.fillStyle = `rgba(${clr},${fade * 0.7})`; xb.fillText(f.isCall ? 'CALL' : 'PUT', baseX + w * 0.36, y); xb.fillStyle = `rgba(255,255,255,${fade * 0.5})`; xb.fillText(f.size.toString(), baseX + w * 0.48, y); xb.fillText(`$${f.premium.toLocaleString()}K`, baseX + w * 0.6, y); if (f.isSweep) { xb.fillStyle = `rgba(255,255,100,${fade * 0.7})`; xb.shadowBlur = 4; xb.shadowColor = 'rgba(255,255,100,0.3)'; xb.fillText('⚡ SWEEP', baseX + w * 0.72, y); xb.shadowBlur = 0; } }); let activeFlows = flows.filter(f => f.y < h); flows.length = 0; flows.push(...activeFlows); rafIds.push(requestAnimationFrame(draw)); }
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s9', start: 'top 60%', onEnter: () => { on = true; addFlow(); const tl = gsap.timeline({ delay: 1 }); tl.to('#s9f', { opacity: 0.15, duration: 0.05 }, 0.5); tl.to('#s9f', { opacity: 0, duration: 0.8 }, 0.55); tl.to('#s9t', { opacity: 1, duration: 0.5 }, 1); tl.to('#s9u', { opacity: 1, duration: 0.8 }, 1.5); tl.to('#s9l', { opacity: 0.12, duration: 0.5 }, 1); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initThePit(); initVolumeProfile(); initOptionsFlow(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="s7">
	<canvas bind:this={s7a}></canvas>
	<canvas bind:this={s7b} class="c2"></canvas>
	<div class="flash" id="s7f"></div>
	<div class="ct" id="s7-content">
		<h1 class="t1" id="s7t">THE PIT</h1>
		<p class="t2" id="s7u">WHERE FORTUNES CHANGE IN SECONDS</p>
	</div>
	<span class="lbl" id="s7l">07 — OPEN OUTCRY</span>
</section>
<div class="sp"></div>

<section class="hero" id="s8">
	<canvas bind:this={s8a}></canvas>
	<canvas bind:this={s8b} class="c2"></canvas>
	<div class="flash" id="s8f"></div>
	<div class="ct" id="s8-content">
		<h1 class="t1" id="s8t">VOLUME PROFILE</h1>
		<p class="t2" id="s8u">WHERE THEY TRADED. NOT JUST WHEN.</p>
	</div>
	<span class="lbl" id="s8l">08 — AUCTION</span>
</section>
<div class="sp"></div>

<section class="hero" id="s9">
	<canvas bind:this={s9a}></canvas>
	<canvas bind:this={s9b} class="c2"></canvas>
	<div class="flash" id="s9f"></div>
	<div class="ct" id="s9-content">
		<h1 class="t1" id="s9t">OPTIONS FLOW</h1>
		<p class="t2" id="s9u">UNUSUAL ACTIVITY. SWEEPS. GREEKS.</p>
	</div>
	<span class="lbl" id="s9l">09 — DERIVATIVES</span>
</section>

<style>
	:global(*) { margin: 0; padding: 0; box-sizing: border-box; }
	:global(html) { overflow-x: hidden; background: #000; }
	.hero { position: relative; width: 100%; height: 100vh; overflow: hidden; display: flex; align-items: center; justify-content: center; color: #fff; font-family: 'JetBrains Mono', monospace; }
	.hero canvas { position: absolute; inset: 0; width: 100%; height: 100%; z-index: 1; }
	.c2 { z-index: 2; }
	.flash { position: absolute; inset: 0; z-index: 30; pointer-events: none; opacity: 0; }
	.ct { position: relative; z-index: 20; text-align: center; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%; pointer-events: none; }
	.lbl { position: absolute; top: 25px; left: 30px; z-index: 60; font-size: 0.38rem; letter-spacing: 0.7em; text-transform: uppercase; opacity: 0.15; }
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(3.5rem, 13vw, 12rem); line-height: 0.85; letter-spacing: -0.01em; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.4rem, 1vw, 0.65rem); letter-spacing: 0.7em; margin-top: 15px; opacity: 0; }
	.sp { height: 2px; background: #000; }
	#s7 { background: #000; } #s7f { background: rgba(255,200,0,0.4); }
	#s7t { background: linear-gradient(180deg, #ffd700, #ff8800); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#s7u { color: rgba(255,200,0,0.5); } #s7l { color: #ffd700; }
	#s8 { background: #000; } #s8f { background: rgba(180,100,255,0.3); }
	#s8t { color: #b464ff; text-shadow: 0 0 80px rgba(180,100,255,0.4); }
	#s8u { color: rgba(180,100,255,0.4); } #s8l { color: #b464ff; }
	#s9 { background: #000; }
	#s9t { color: #ff00cc; text-shadow: 0 0 80px rgba(255,0,200,0.4); }
	#s9u { color: rgba(255,0,200,0.4); } #s9l { color: #ff00cc; }
</style>
