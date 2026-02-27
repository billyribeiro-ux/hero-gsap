<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let s4a: HTMLCanvasElement, s4b: HTMLCanvasElement;
	let s5a: HTMLCanvasElement, s5b: HTMLCanvasElement;
	let s6a: HTMLCanvasElement, s6b: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }
	function shake(el: HTMLElement, intensity = 20, dur = 0.5) { const s = Math.floor(dur / 0.016); for (let i = 0; i < s; i++) { const f = Math.pow(1 - i / s, 2); gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.6, delay: i * 0.016 }); } gsap.set(el, { x: 0, y: 0, delay: s * 0.016 }); }

	function initTheTape() {
		const cv = [s4a, s4b];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s4a.width = s4a.parentElement!.offsetWidth, h = s4a.height = s4a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const entries: { col: number; sym: string; price: string; size: number; side: boolean; y: number; speed: number; isBlock: boolean; alpha: number }[] = [];
		const blocks: { y: number; alpha: number; time: number }[] = [];
		const SYMS = ['ES', 'NQ', 'SPX', 'AAPL', 'TSLA', 'NVDA', 'AMZN', 'MSFT'];
		function addEntry() { if (!on) return; const isBlock = Math.random() > 0.93; const sym = SYMS[Math.floor(Math.random() * SYMS.length)]; const size = isBlock ? Math.floor(Math.random() * 5000 + 500) : Math.floor(Math.random() * 100 + 1); const price = (5890 + Math.random() * 20).toFixed(2); const side = Math.random() > 0.5; const col = Math.floor(Math.random() * 4); entries.push({ col, sym, price, size, side, y: -25, speed: 3 + Math.random() * 4, isBlock, alpha: isBlock ? 1 : 0.5 }); if (isBlock) blocks.push({ y: 0, alpha: 0.4, time: t }); timeouts.push(window.setTimeout(addEntry, isBlock ? 300 : 20 + Math.random() * 30)); }
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s4a.width; h = s4a.height; }
		window.addEventListener('resize', resize);
		function draw() { if (!on) { rafIds.push(requestAnimationFrame(draw)); return; } t += 0.01; const [xa, xb] = ctx; xa.fillStyle = 'rgba(0,0,0,0.1)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xb.fillStyle = 'rgba(255,255,255,0.1)'; xb.font = '8px Orbitron'; ['SYMBOL', 'PRICE', 'SIZE', 'SIDE'].forEach((h2, i) => { xb.fillText(h2, w * 0.15 + i * w * 0.18, 20); }); xb.strokeStyle = 'rgba(255,255,255,0.05)'; xb.beginPath(); xb.moveTo(w * 0.1, 28); xb.lineTo(w * 0.9, 28); xb.stroke(); entries.forEach(e => { e.y += e.speed; const baseX = w * 0.15; const y = e.y + 30; const clr = e.side ? '0,255,136' : '255,51,68'; const sz = e.isBlock ? 'bold 12' : '10'; if (e.isBlock && e.y > 0) { xa.fillStyle = `rgba(${clr},${Math.max(0, (0.5 - e.y / h) * 0.08)})`; xa.fillRect(w * 0.08, y - 8, w * 0.84, 16); } xb.font = `${sz}px "JetBrains Mono"`; xb.fillStyle = `rgba(${clr},${e.alpha * (1 - e.y / (h * 1.5))})`; xb.fillText(e.sym, baseX, y); xb.fillText(e.price, baseX + w * 0.18, y); xb.fillText(e.size.toLocaleString(), baseX + w * 0.36, y); xb.fillText(e.side ? 'BID' : 'ASK', baseX + w * 0.54, y); if (e.isBlock) { xb.fillStyle = `rgba(255,255,100,${(1 - e.y / (h * 1.5)) * 0.6})`; xb.shadowBlur = 6; xb.shadowColor = 'rgba(255,255,100,0.3)'; xb.fillText('■ BLOCK', baseX + w * 0.65, y); xb.shadowBlur = 0; } }); let activeEntries = entries.filter(e => e.y < h); entries.length = 0; entries.push(...activeEntries); blocks.forEach(b => { b.alpha *= 0.97; b.y += 6; xa.strokeStyle = `rgba(255,255,100,${b.alpha})`; xa.lineWidth = 0.5; xa.beginPath(); xa.moveTo(w * 0.08, b.y); xa.lineTo(w * 0.92, b.y); xa.stroke(); }); let activeBlocks = blocks.filter(b => b.alpha > 0.01); blocks.length = 0; blocks.push(...activeBlocks); rafIds.push(requestAnimationFrame(draw)); }
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s4', start: 'top 60%', onEnter: () => { on = true; addEntry(); const tl = gsap.timeline({ delay: 1 }); tl.to('#s4t', { opacity: 1, duration: 0.5 }, 1); tl.to('#s4u', { opacity: 1, duration: 0.8 }, 1.5); tl.to('#s4l', { opacity: 0.12, duration: 0.5 }, 1); } }); triggers.push(trigger);
	}

	function initDarkPool() {
		const cv = [s5a, s5b];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s5a.width = s5a.parentElement!.offsetWidth, h = s5a.height = s5a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const rings: { x: number; y: number; r: number; a: number }[] = [];
		const icebergs: { x: number; y: number; visibleSize: number; hiddenSize: number; volume: number; side: string; revealed: number }[] = [];
		for (let i = 0; i < 12; i++) { icebergs.push({ x: w * 0.15 + Math.random() * w * 0.7, y: h * 0.15 + Math.random() * h * 0.7, visibleSize: Math.random() * 15 + 5, hiddenSize: Math.random() * 80 + 40, volume: Math.floor(Math.random() * 80000 + 10000), side: Math.random() > 0.5 ? 'BID' : 'ASK', revealed: 0 }); }
		const particles: { x: number; y: number; vx: number; vy: number; s: number }[] = [];
		for (let i = 0; i < 200; i++) particles.push({ x: Math.random() * 2000, y: Math.random() * 1200, vx: (Math.random() - 0.5) * 0.3, vy: (Math.random() - 0.5) * 0.3, s: Math.random() * 1.5 + 0.3 });
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s5a.width; h = s5a.height; }
		window.addEventListener('resize', resize);
		function ping() { if (!on) return; rings.push({ x: mx, y: my, r: 0, a: 0.5 }); icebergs.forEach(ice => { if (Math.hypot(ice.x - mx, ice.y - my) < 300) gsap.to(ice, { revealed: Math.min(1, ice.revealed + 0.25), duration: 0.8, ease: 'power2.out' }); }); timeouts.push(window.setTimeout(ping, 2500)); }
		function draw() { if (!on) { rafIds.push(requestAnimationFrame(draw)); return; } t += 0.006; const [xa, xb] = ctx; xa.fillStyle = 'rgba(0,3,8,0.05)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); particles.forEach(p => { p.x += p.vx; p.y += p.vy; if (p.x < 0) p.x = w; if (p.x > w) p.x = 0; if (p.y < 0) p.y = h; if (p.y > h) p.y = 0; xa.beginPath(); xa.arc(p.x / 2000 * w, p.y / 1200 * h, p.s, 0, Math.PI * 2); xa.fillStyle = 'rgba(26,107,255,0.08)'; xa.fill(); }); rings.forEach(r => { r.r += 4; r.a *= 0.97; xb.beginPath(); xb.arc(r.x, r.y, r.r, 0, Math.PI * 2); xb.strokeStyle = `rgba(26,107,255,${r.a})`; xb.lineWidth = 2; xb.stroke(); }); let activeRings = rings.filter(r => r.a > 0.01); rings.length = 0; rings.push(...activeRings); icebergs.forEach(ice => { if (ice.revealed < 0.02) return; const a = ice.revealed; const clr = ice.side === 'BID' ? '0,200,136' : '200,80,100'; xb.fillStyle = `rgba(${clr},${0.15 + a * 0.2})`; xb.fillRect(ice.x - ice.visibleSize / 2, ice.y - ice.visibleSize / 4, ice.visibleSize, ice.visibleSize / 2); xb.fillStyle = `rgba(${clr},${a * 0.12})`; xb.fillRect(ice.x - ice.hiddenSize / 2 * a, ice.y + ice.visibleSize / 4, ice.hiddenSize * a, ice.hiddenSize * 0.6 * a); xb.strokeStyle = `rgba(${clr},${a * 0.2})`; xb.lineWidth = 1; xb.strokeRect(ice.x - ice.hiddenSize / 2 * a, ice.y + ice.visibleSize / 4, ice.hiddenSize * a, ice.hiddenSize * 0.6 * a); xb.fillStyle = `rgba(${clr},${a * 0.6})`; xb.font = 'bold 9px "JetBrains Mono"'; xb.textAlign = 'center'; xb.fillText(`${ice.side} ${ice.volume.toLocaleString()}`, ice.x, ice.y + ice.hiddenSize * 0.3 * a + ice.visibleSize); xb.fillStyle = `rgba(${clr},${a * 0.3})`; xb.font = '7px "JetBrains Mono"'; xb.fillText('HIDDEN ORDER', ice.x, ice.y + ice.hiddenSize * 0.3 * a + ice.visibleSize + 12); xb.textAlign = 'start'; const grd = xb.createRadialGradient(ice.x, ice.y, 0, ice.x, ice.y, ice.hiddenSize * a); grd.addColorStop(0, `rgba(26,107,255,${a * 0.04})`); grd.addColorStop(1, 'transparent'); xb.fillStyle = grd; xb.fillRect(ice.x - ice.hiddenSize * a, ice.y - ice.hiddenSize * a, ice.hiddenSize * 2 * a, ice.hiddenSize * 2 * a); }); rafIds.push(requestAnimationFrame(draw)); }
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s5', start: 'top 60%', onEnter: () => { on = true; ping(); const tl = gsap.timeline({ delay: 1 }); tl.to('#s5t', { opacity: 1, duration: 0.8 }, 2); tl.to('#s5u', { opacity: 1, duration: 1 }, 2.8); tl.to('#s5l', { opacity: 0.12, duration: 0.5 }, 2); } }); triggers.push(trigger);
	}

	function initPriceAction() {
		const cv = [s6a, s6b];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s6a.width = s6a.parentElement!.offsetWidth, h = s6a.height = s6a.parentElement!.offsetHeight;
		let t = 0, on = false, buildIdx = 0;
		const candles: { o: number; c: number; h: number; l: number; bull: boolean }[] = [];
		const swingPoints: { idx: number; type: string; price: number }[] = [];
		let price = 5870;
		for (let i = 0; i < 45; i++) { const o = price; if (i % 8 < 5) { price += Math.random() * 6 + 1; } else { price -= Math.random() * 8 + 2; } const c = price; candles.push({ o, c, h: Math.max(o, c) + Math.random() * 4, l: Math.min(o, c) - Math.random() * 4, bull: c >= o }); if (i % 8 === 4) swingPoints.push({ idx: i, type: 'high', price: Math.max(o, c) }); if (i % 8 === 7) swingPoints.push({ idx: i, type: 'low', price: Math.min(o, c) }); }
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s6a.width; h = s6a.height; }
		window.addEventListener('resize', resize);
		function draw() { if (!on) { rafIds.push(requestAnimationFrame(draw)); return; } t += 0.008; const [xa, xb] = ctx; xa.clearRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xa.strokeStyle = 'rgba(255,255,255,0.02)'; xa.lineWidth = 0.5; for (let x = 0; x < w; x += w / 20) { xa.beginPath(); xa.moveTo(x, 0); xa.lineTo(x, h); xa.stroke(); } for (let y = 0; y < h; y += h / 12) { xa.beginPath(); xa.moveTo(0, y); xa.lineTo(w, y); xa.stroke(); } const vis = candles.slice(0, buildIdx); if (!vis.length) { rafIds.push(requestAnimationFrame(draw)); return; } const ps = vis.flatMap(c => [c.h, c.l]); const minP = Math.min(...ps) - 8, maxP = Math.max(...ps) + 8, range = maxP - minP; const gap = w * 0.8 / candles.length, cW = Math.min(gap * 0.55, 9), offX = w * 0.1; vis.forEach((c, idx) => { const i = candles.indexOf(c); const x = offX + i * gap + gap / 2; const oY = h * 0.08 + (maxP - c.o) / range * h * 0.8; const cY = h * 0.08 + (maxP - c.c) / range * h * 0.8; const hY = h * 0.08 + (maxP - c.h) / range * h * 0.8; const lY = h * 0.08 + (maxP - c.l) / range * h * 0.8; xb.strokeStyle = c.bull ? 'rgba(0,255,136,0.6)' : 'rgba(255,51,68,0.6)'; xb.lineWidth = 1; xb.beginPath(); xb.moveTo(x, hY); xb.lineTo(x, lY); xb.stroke(); xb.fillStyle = c.bull ? 'rgba(0,255,136,0.8)' : 'rgba(255,51,68,0.8)'; xb.fillRect(x - cW / 2, Math.min(oY, cY), cW, Math.abs(cY - oY) || 1.5); }); const lows = swingPoints.filter(s => s.type === 'low' && s.idx < buildIdx); if (lows.length >= 2) { xb.beginPath(); lows.forEach((s, i) => { const x = offX + s.idx * gap + gap / 2; const y = h * 0.08 + (maxP - s.price) / range * h * 0.8; i === 0 ? xb.moveTo(x, y) : xb.lineTo(x, y); }); xb.strokeStyle = 'rgba(0,255,136,0.2)'; xb.lineWidth = 1.5; xb.setLineDash([6, 3]); xb.stroke(); xb.setLineDash([]); } swingPoints.forEach(s => { if (s.idx >= buildIdx) return; const x = offX + s.idx * gap + gap / 2; const y = h * 0.08 + (maxP - s.price) / range * h * 0.8; xb.fillStyle = s.type === 'high' ? 'rgba(0,255,136,0.25)' : 'rgba(255,200,50,0.25)'; xb.font = '7px "JetBrains Mono"'; xb.textAlign = 'center'; xb.fillText(s.type === 'high' ? 'HH' : 'HL', x, s.type === 'high' ? y - 10 : y + 14); xb.beginPath(); xb.arc(x, y, 3, 0, Math.PI * 2); xb.fillStyle = s.type === 'high' ? 'rgba(0,255,136,0.3)' : 'rgba(255,200,50,0.3)'; xb.fill(); xb.textAlign = 'start'; }); rafIds.push(requestAnimationFrame(draw)); }
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s6', start: 'top 60%', onEnter: () => { on = true; candles.forEach((c, i) => { timeouts.push(window.setTimeout(() => { buildIdx = i + 1; }, i * 60)); }); const endT = candles.length * 0.06 + 0.3; const tl = gsap.timeline(); tl.to('#s6f', { opacity: 0.2, duration: 0.05 }, endT); tl.to('#s6f', { opacity: 0, duration: 1 }, endT + 0.05); tl.to('#s6t', { opacity: 1, duration: 0.5 }, endT); tl.to('#s6u', { opacity: 1, duration: 1 }, endT + 0.5); tl.to('#s6l', { opacity: 0.12, duration: 0.5 }, endT + 0.2); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initTheTape(); initDarkPool(); initPriceAction(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="s4">
	<canvas bind:this={s4a}></canvas>
	<canvas bind:this={s4b} class="c2"></canvas>
	<div class="flash" id="s4f"></div>
	<div class="ct" id="s4-content">
		<h1 class="t1" id="s4t">THE TAPE</h1>
		<p class="t2" id="s4u">THE TAPE DOESN'T LIE</p>
	</div>
	<span class="lbl" id="s4l">04 — TIME & SALES</span>
</section>
<div class="sp"></div>

<section class="hero" id="s5">
	<canvas bind:this={s5a}></canvas>
	<canvas bind:this={s5b} class="c2"></canvas>
	<div class="ct" id="s5-content">
		<h1 class="t1" id="s5t">DARK POOL</h1>
		<p class="t2" id="s5u">HIDDEN LIQUIDITY. INSTITUTIONAL INTENT.</p>
	</div>
	<span class="lbl" id="s5l">05 — OFF-EXCHANGE</span>
</section>
<div class="sp"></div>

<section class="hero" id="s6">
	<canvas bind:this={s6a}></canvas>
	<canvas bind:this={s6b} class="c2"></canvas>
	<div class="flash" id="s6f"></div>
	<div class="ct" id="s6-content">
		<h1 class="t1" id="s6t">PRICE ACTION</h1>
		<p class="t2" id="s6u">NO INDICATORS. NO NOISE. JUST TRUTH.</p>
	</div>
	<span class="lbl" id="s6l">06 — PURE CHART</span>
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
	#s4 { background: #000; } #s4f { background: rgba(0,255,136,0.2); }
	#s4t { color: #00ff88; text-shadow: 0 0 60px rgba(0,255,136,0.4); }
	#s4u { color: rgba(0,255,136,0.4); } #s4l { color: #00ff88; }
	#s5 { background: #000; }
	#s5t { color: #1a6bff; text-shadow: 0 0 100px rgba(26,107,255,0.5); }
	#s5u { color: rgba(26,107,255,0.4); } #s5l { color: #1a6bff; }
	#s6 { background: #000; } #s6f { background: rgba(255,255,255,0.3); }
	#s6t { color: #fff; text-shadow: 0 0 40px rgba(255,255,255,0.3); }
	#s6u { color: rgba(255,255,255,0.4); } #s6l { color: #888; }
</style>
