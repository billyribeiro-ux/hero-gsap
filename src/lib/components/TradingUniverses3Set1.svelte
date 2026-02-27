<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0, prevMx = 0, prevMy = 0, mvx = 0, mvy = 0;
	let h1c1: HTMLCanvasElement, h1c2: HTMLCanvasElement, h1c3: HTMLCanvasElement;
	let h2c1: HTMLCanvasElement, h2c2: HTMLCanvasElement;
	let h3c1: HTMLCanvasElement, h3c2: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }
	function updateMouseVelocity() { mvx = mx - prevMx; mvy = my - prevMy; prevMx = mx; prevMy = my; rafIds.push(requestAnimationFrame(updateMouseVelocity)); }
	function shake(el: HTMLElement, intensity = 20, dur = 0.5) { const s = Math.floor(dur / 0.02); for (let j = 0; j < s; j++) { const f = 1 - j / s; gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.7, delay: j * 0.02 }); } gsap.set(el, { x: 0, y: 0, delay: s * 0.02 }); }

	function initFloor() {
		const x1 = h1c1.getContext('2d')!, x2 = h1c2.getContext('2d')!, x3 = h1c3.getContext('2d')!;
		let w = h1c1.width = h1c1.parentElement!.offsetWidth, h = h1c1.height = h1c1.parentElement!.offsetHeight;
		let t = 0, active = false; const GRID = 20, cols = Math.ceil(1920 / GRID), rows = Math.ceil(1080 / GRID), energyField: number[] = new Array(cols * rows).fill(0);
		const SYMS = ['ES', 'NQ', 'SPX', 'AAPL', 'TSLA', 'NVDA', 'GC', 'CL', 'ZB', 'VIX'];
		const tickets: { x: number; y: number; vx: number; vy: number; text: string; size: number; alpha: number; rot: number; color: string }[] = [];
		for (let i = 0; i < 300; i++) { const a = Math.random() * Math.PI * 2, spd = Math.random() * 4 + 1; tickets.push({ x: Math.random() * 1920, y: Math.random() * 1080, vx: Math.cos(a) * spd, vy: Math.sin(a) * spd, text: SYMS[Math.floor(Math.random() * SYMS.length)] + ' ' + (Math.random() > 0.5 ? 'BUY' : 'SELL'), size: Math.random() * 8 + 6, alpha: Math.random() * 0.3 + 0.05, rot: (Math.random() - 0.5) * 0.5, color: Math.random() > 0.5 ? '0,255,136' : '255,51,68' }); }
		const waveData: number[] = new Array(200).fill(0);
		function resize() { [h1c1, h1c2, h1c3].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h1c1.width; h = h1c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; x1.fillStyle = 'rgba(2,1,0,0.05)'; x1.fillRect(0, 0, w, h);
			const mcol = Math.floor(mx / GRID), mrow = Math.floor(my / GRID); for (let dr = -5; dr <= 5; dr++) for (let dc = -5; dc <= 5; dc++) { const r = mrow + dr, c = mcol + dc; if (r >= 0 && r < rows && c >= 0 && c < cols) { const dist = Math.sqrt(dr * dr + dc * dc); energyField[r * cols + c] = Math.min(1, energyField[r * cols + c] + Math.max(0, (1 - dist / 5) * 0.1)); } }
			for (let i = 0; i < energyField.length; i++) { energyField[i] *= 0.98; if (energyField[i] > 0.01) { const c = i % cols, r = Math.floor(i / cols), v = energyField[i]; x1.fillStyle = `rgba(255,${Math.floor(100 + v * 100)},0,${v * 0.15})`; x1.fillRect(c * GRID, r * GRID, GRID, GRID); } }
			x2.clearRect(0, 0, w, h); tickets.forEach(tk => { const dx = mx - tk.x, dy = my - tk.y, dist = Math.sqrt(dx * dx + dy * dy); if (dist < 200) { const force = (1 - dist / 200) * 2; tk.vx -= (dx / dist) * force * 0.3; tk.vy -= (dy / dist) * force * 0.3; } tk.vx += (Math.random() - 0.5) * 0.2; tk.vy += (Math.random() - 0.5) * 0.2; tk.vx *= 0.99; tk.vy *= 0.99; tk.x += tk.vx; tk.y += tk.vy; if (tk.x < -100) tk.x = w + 100; if (tk.x > w + 100) tk.x = -100; if (tk.y < -50) tk.y = h + 50; if (tk.y > h + 50) tk.y = -50; x2.save(); x2.translate(tk.x, tk.y); x2.rotate(tk.rot + Math.sin(t + tk.x * 0.01) * 0.1); x2.font = `${tk.size}px "JetBrains Mono"`; x2.fillStyle = `rgba(${tk.color},${tk.alpha})`; x2.fillText(tk.text, 0, 0); x2.restore(); });
			x3.clearRect(0, 0, w, h); waveData.shift(); waveData.push(Math.sin(t * 15) * 30 + Math.sin(t * 23) * 20 + Math.sin(t * 7) * 40 + (Math.random() - 0.5) * 60); x3.beginPath(); const baseY = h * 0.85; waveData.forEach((v, i) => { const px = (i / waveData.length) * w; if (i === 0) x3.moveTo(px, baseY + v); else x3.lineTo(px, baseY + v); }); x3.strokeStyle = 'rgba(255,200,0,0.15)'; x3.lineWidth = 2; x3.shadowBlur = 15; x3.shadowColor = 'rgba(255,200,0,0.3)'; x3.stroke(); x3.shadowBlur = 0;
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h1', start: 'top 70%', onEnter: () => { active = true; const tl = gsap.timeline({ delay: 0.4 }); tl.to('#h1f', { opacity: 0.6, duration: 0.08 }, 0.5); tl.to('#h1f', { opacity: 0, duration: 1 }, 0.6); tl.to('#h1t', { opacity: 1, duration: 0.6, ease: 'power4.out' }, 0.5); tl.add(() => shake(document.getElementById('h1-content')!, 25, 0.5), 0.5); tl.to('#h1s', { opacity: 1, duration: 0.8 }, 1.2); tl.to('#h1s2', { opacity: 1, duration: 0.8 }, 1.6); } }); triggers.push(trigger);
	}

	function initAlgorithm() {
		const x1 = h2c1.getContext('2d')!, x2 = h2c2.getContext('2d')!; let w = h2c1.width = h2c1.parentElement!.offsetWidth, h = h2c1.height = h2c1.parentElement!.offsetHeight;
		let t = 0, active = false, compiling = false; const chars = '01', streams: { x: number; y: number; speed: number; chars: string[]; len: number }[] = [];
		for (let i = 0; i < 40; i++) streams.push({ x: Math.random() * w, y: Math.random() * h * 0.6, speed: Math.random() * 2 + 1, chars: Array(20).fill(0).map(() => chars[Math.floor(Math.random() * 2)]), len: Math.floor(Math.random() * 15 + 5) });
		function resize() { [h2c1, h2c2].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h2c1.width; h = h2c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.01; x1.fillStyle = compiling ? 'rgba(0,10,5,0.25)' : 'rgba(0,10,5,0.08)'; x1.fillRect(0, 0, w, h);
			x1.font = '12px "JetBrains Mono"'; streams.forEach(s => { if (Math.random() > 0.95) s.chars.pop(); if (Math.random() > 0.95) s.chars.unshift(chars[Math.floor(Math.random() * 2)]); if (s.chars.length > s.len) s.chars.length = s.len; s.y += s.speed; if (s.y > h + 100) { s.y = -50; s.x = Math.random() * w; } const grd = x1.createLinearGradient(0, s.y - s.chars.length * 15, 0, s.y); grd.addColorStop(0, 'transparent'); grd.addColorStop(0.5, `rgba(0,255,136,${0.2 + Math.sin(t * 3) * 0.1})`); grd.addColorStop(1, 'rgba(0,255,136,0.6)'); x1.fillStyle = grd; x1.textAlign = 'center'; s.chars.forEach((c, i) => x1.fillText(c, s.x, s.y - i * 15)); x1.textAlign = 'start'; });
			x2.clearRect(0, 0, w, h); x2.font = '10px "JetBrains Mono"'; const pct = Math.floor((Math.sin(t) + 1) * 40 + 10); x2.fillStyle = `rgba(0,255,136,${0.1 + Math.sin(t * 2) * 0.05})`; x2.fillText(`COMPILING ${pct}%`, 30, h - 30); x2.fillText(`ERRORS: 0`, 30, h - 50);
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h2', start: 'top 70%', onEnter: () => { active = true; const tl = gsap.timeline({ delay: 0.3 }); tl.to('#h2f', { opacity: 1, duration: 0.15 }, 0.4); tl.add(() => { compiling = true; }, 0.4); tl.to('#h2f', { opacity: 0, duration: 0.8 }, 0.55); tl.add(() => { compiling = false; }, 1.5); tl.to('#h2t', { opacity: 1, duration: 0.6, ease: 'power4.out' }, 0.8); tl.add(() => shake(document.getElementById('h2-content')!, 20, 0.4), 0.8); tl.to('#h2s', { opacity: 1, duration: 0.8 }, 1.5); tl.to('#h2s2', { opacity: 1, duration: 0.8 }, 1.9); } }); triggers.push(trigger);
	}

	function initDarkPool() {
		const x1 = h3c1.getContext('2d')!, x2 = h3c2.getContext('2d')!; let w = h3c1.width = h3c1.parentElement!.offsetWidth, h = h3c1.height = h3c1.parentElement!.offsetHeight;
		let t = 0, active = false; const orders: { x: number; y: number; size: number; opacity: number; vx: number; vy: number }[] = [];
		for (let i = 0; i < 50; i++) orders.push({ x: Math.random() * w, y: Math.random() * h, size: Math.random() * 100 + 50, opacity: 0, vx: (Math.random() - 0.5) * 0.3, vy: (Math.random() - 0.5) * 0.3 });
		function resize() { [h3c1, h3c2].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h3c1.width; h = h3c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.005; x1.fillStyle = 'rgba(0,3,8,0.04)'; x1.fillRect(0, 0, w, h);
			orders.forEach(o => { o.x += o.vx; o.y += o.vy; if (Math.random() > 0.99) { o.opacity += 0.05; if (o.opacity > 0.3) o.opacity = 0; } const grd = x1.createRadialGradient(o.x, o.y, 0, o.x, o.y, o.size); grd.addColorStop(0, `rgba(26,107,255,${o.opacity * 0.3})`); grd.addColorStop(1, 'transparent'); x1.fillStyle = grd; x1.fillRect(o.x - o.size, o.y - o.size, o.size * 2, o.size * 2); });
			x2.clearRect(0, 0, w, h); x2.strokeStyle = `rgba(26,107,255,${0.1 + Math.sin(t) * 0.05})`; x2.lineWidth = 1; x2.setLineDash([5, 5]); x2.beginPath(); x2.moveTo(mx, 0); x2.lineTo(mx, h); x2.stroke(); x2.beginPath(); x2.moveTo(0, my); x2.lineTo(w, my); x2.stroke(); x2.setLineDash([]); x2.fillStyle = 'rgba(26,107,255,0.2)'; x2.font = '10px "JetBrains Mono"'; x2.fillText(`ZONE: ${Math.floor(mx / w * 100)}:${Math.floor(my / h * 100)}`, mx + 10, my - 10);
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h3', start: 'top 70%', onEnter: () => { active = true; const tl = gsap.timeline({ delay: 0.5 }); tl.to('#h3t', { opacity: 1, duration: 0.8, ease: 'power4.out' }, 0.5); tl.add(() => shake(document.getElementById('h3-content')!, 15, 0.3), 0.5); tl.to('#h3s', { opacity: 1, duration: 0.8 }, 1.3); tl.to('#h3s2', { opacity: 1, duration: 0.8 }, 1.7); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; prevMx = mx; prevMy = my; rafIds.push(requestAnimationFrame(updateMouseVelocity)); initFloor(); initAlgorithm(); initDarkPool(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="h1" style="background:#020100">
	<canvas bind:this={h1c1}></canvas>
	<canvas bind:this={h1c2} class="c2"></canvas>
	<canvas bind:this={h1c3} class="c3"></canvas>
	<div class="vig h1-vig"></div>
	<div class="grn"></div>
	<div class="flash" id="h1f" style="background:radial-gradient(circle,rgba(255,200,50,0.8),transparent 60%)"></div>
	<div class="ct" id="h1-content">
		<h1 class="t1" id="h1t">THE FLOOR</h1>
		<p class="t2" id="h1s">WHERE FORTUNES ARE MADE IN SECONDS</p>
		<p class="t3" id="h1s2">[ OPEN OUTCRY — CHICAGO, 1987 ]</p>
	</div>
	<span class="lbl h1-lbl">01 — THE FLOOR</span>
	<span class="idx h1-idx">01</span>
</section>
<div class="sp"></div>

<section class="hero" id="h2" style="background:#000a05">
	<canvas bind:this={h2c1}></canvas>
	<canvas bind:this={h2c2} class="c2"></canvas>
	<div class="vig h2-vig"></div>
	<div class="grn"></div>
	<div class="flash" id="h2f" style="background:#00ff88"></div>
	<div class="ct" id="h2-content">
		<h1 class="t1" id="h2t">ALGORITHM</h1>
		<p class="t2" id="h2s">LOGIC IS THE ULTIMATE WEAPON</p>
		<p class="t3" id="h2s2">[ LATENCY: 0.003ms // EXECUTING ]</p>
	</div>
	<span class="lbl h2-lbl">02 — ALGORITHM</span>
	<span class="idx h2-idx">02</span>
</section>
<div class="sp"></div>

<section class="hero" id="h3" style="background:#000308">
	<canvas bind:this={h3c1}></canvas>
	<canvas bind:this={h3c2} class="c2"></canvas>
	<div class="vig h3-vig"></div>
	<div class="grn"></div>
	<div class="ct" id="h3-content">
		<h1 class="t1" id="h3t">DARK POOL</h1>
		<p class="t2" id="h3s">THE ORDERS YOU'LL NEVER SEE COMING</p>
		<p class="t3" id="h3s2">[ HIDDEN LIQUIDITY DETECTED ]</p>
	</div>
	<span class="lbl h3-lbl">03 — DARK POOL</span>
	<span class="idx h3-idx">03</span>
</section>

<style>
	:global(*) { margin: 0; padding: 0; box-sizing: border-box; }
	:global(html) { overflow-x: hidden; background: #000; }
	.hero { position: relative; width: 100%; height: 100vh; overflow: hidden; display: flex; align-items: center; justify-content: center; color: #fff; font-family: 'JetBrains Mono', monospace; }
	.hero canvas { position: absolute; inset: 0; width: 100%; height: 100%; z-index: 1; }
	.c2 { z-index: 2; }
	.c3 { z-index: 3; }
	.vig { position: absolute; inset: 0; z-index: 40; pointer-events: none; }
	.grn { position: absolute; inset: 0; z-index: 50; pointer-events: none; opacity: 0.03; mix-blend-mode: overlay; background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E"); background-size: 128px; }
	.flash { position: absolute; inset: 0; z-index: 30; pointer-events: none; opacity: 0; }
	.ct { position: relative; z-index: 10; text-align: center; pointer-events: none; }
	.lbl { position: absolute; top: 25px; left: 30px; z-index: 60; font-size: 0.45rem; letter-spacing: 0.6em; text-transform: uppercase; opacity: 0.15; }
	.idx { position: absolute; bottom: 25px; right: 35px; z-index: 60; font-family: 'Bebas Neue'; font-size: 5rem; line-height: 1; opacity: 0.03; }
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(3.5rem, 13vw, 11rem); line-height: 0.85; letter-spacing: 0.02em; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.45rem, 1.1vw, 0.7rem); letter-spacing: 0.7em; margin-top: 18px; opacity: 0; }
	.t3 { font-family: 'JetBrains Mono', monospace; font-size: 0.5rem; letter-spacing: 0.35em; margin-top: 28px; opacity: 0; }
	.sp { height: 2px; background: #000; }
	.h1-vig { background: radial-gradient(ellipse at 50% 60%, transparent 20%, rgba(2,1,0,0.85) 100%); }
	.h2-vig { background: radial-gradient(ellipse, transparent 30%, rgba(0,10,5,0.85) 100%); }
	.h3-vig { background: radial-gradient(ellipse, transparent 25%, rgba(0,3,8,0.9) 100%); }
	#h1t { background: linear-gradient(180deg, #ffd700 0%, #ff8c00 50%, #8b4513 100%); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#h1s { color: #ffd700; }
	#h1s2 { color: rgba(255,200,0,0.3); }
	#h2t { color: #00ff88; text-shadow: 0 0 80px rgba(0,255,136,0.3); }
	#h2s { color: rgba(0,255,136,0.5); }
	#h2s2 { color: rgba(0,255,136,0.2); }
	#h3t { color: #1a6bff; text-shadow: 0 0 100px rgba(26,107,255,0.4); }
	#h3s { color: rgba(26,107,255,0.4); }
	#h3s2 { color: rgba(26,107,255,0.2); }
	.h1-lbl, .h1-idx { color: #ffd700; }
	.h2-lbl, .h2-idx { color: #00ff88; }
	.h3-lbl, .h3-idx { color: #1a6bff; }
</style>
