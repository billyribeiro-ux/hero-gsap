<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let h4c1: HTMLCanvasElement, h4c2: HTMLCanvasElement;
	let h5c1: HTMLCanvasElement, h5c2: HTMLCanvasElement;
	let h6c1: HTMLCanvasElement, h6c2: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }
	function shake(el: HTMLElement, intensity = 20, dur = 0.5) { const s = Math.floor(dur / 0.02); for (let j = 0; j < s; j++) { const f = 1 - j / s; gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.7, delay: j * 0.02 }); } gsap.set(el, { x: 0, y: 0, delay: s * 0.02 }); }

	function initMomentum() {
		const x1 = h4c1.getContext('2d')!, x2 = h4c2.getContext('2d')!; let w = h4c1.width = h4c1.parentElement!.offsetWidth, h = h4c1.height = h4c1.parentElement!.offsetHeight;
		let t = 0, active = false; const ball = { x: 0, y: h * 0.5, vx: 8, vy: 0, r: 20 }, sparks: { x: number; y: number; vx: number; vy: number; life: number }[] = [];
		function spawnSpark(bx: number, by: number) { for (let i = 0; i < 5; i++) sparks.push({ x: bx, y: by, vx: (Math.random() - 0.5) * 10, vy: (Math.random() - 0.5) * 10, life: 1 }); }
		function resize() { [h4c1, h4c2].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h4c1.width; h = h4c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.016; x1.fillStyle = 'rgba(5,2,0,0.1)'; x1.fillRect(0, 0, w, h);
			x1.fillStyle = 'rgba(255,255,255,0.05)'; for (let i = 0; i < 10; i++) { const lx = w * 0.15 + i * w * 0.09; x1.fillRect(lx, 0, 1, h); }
			ball.x += ball.vx; ball.y += ball.vy; ball.vy += 0.3;
			if (ball.y + ball.r > h * 0.8) { ball.y = h * 0.8 - ball.r; ball.vy *= -0.7; spawnSpark(ball.x, ball.y + ball.r); }
			if (ball.x > w * 0.85) { ball.vx *= -0.8; ball.x = w * 0.85; spawnSpark(ball.x, ball.y); }
			if (ball.x < w * 0.15) { ball.x = w * 0.15; ball.vx = 8; ball.y = h * 0.5; ball.vy = 0; }
			const grd = x1.createRadialGradient(ball.x, ball.y, 0, ball.x, ball.y, ball.r * 2); grd.addColorStop(0, 'rgba(0,255,136,0.8)'); grd.addColorStop(1, 'transparent'); x1.fillStyle = grd; x1.beginPath(); x1.arc(ball.x, ball.y, ball.r * 2, 0, Math.PI * 2); x1.fill(); x1.fillStyle = '#fff'; x1.beginPath(); x1.arc(ball.x, ball.y, ball.r * 0.5, 0, Math.PI * 2); x1.fill();
			x2.clearRect(0, 0, w, h); sparks.forEach((s, i) => { s.x += s.vx; s.y += s.vy; s.vy += 0.5; s.life -= 0.02; x2.beginPath(); x2.arc(s.x, s.y, 2 * s.life, 0, Math.PI * 2); x2.fillStyle = `rgba(0,255,136,${s.life})`; x2.fill(); }); sparks.filter(s => s.life > 0);
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h4', start: 'top 70%', onEnter: () => { active = true; const tl = gsap.timeline({ delay: 0.3 }); tl.to('#h4f', { opacity: 0.8, duration: 0.05 }, 0.4); tl.to('#h4f', { opacity: 0, duration: 1.5 }, 0.45); tl.to('#h4t', { opacity: 1, duration: 0.6, ease: 'power4.out' }, 0.8); tl.add(() => shake(document.getElementById('h4-content')!, 30, 0.5), 0.8); tl.to('#h4s', { opacity: 1, duration: 0.8 }, 1.5); } }); triggers.push(trigger);
	}

	function initSpread() {
		const x1 = h5c1.getContext('2d')!, x2 = h5c2.getContext('2d')!; let w = h5c1.width = h5c1.parentElement!.offsetWidth, h = h5c1.height = h5c1.parentElement!.offsetHeight;
		let t = 0, active = false; const bids: { x: number; y: number; size: number; speed: number }[] = [], asks: { x: number; y: number; size: number; speed: number }[] = [], sparks: { x: number; y: number; vx: number; vy: number; life: number; color: string }[] = [];
		for (let i = 0; i < 30; i++) { bids.push({ x: Math.random() * w * 0.4, y: Math.random() * h, size: Math.random() * 3 + 2, speed: Math.random() * 2 + 1 }); asks.push({ x: w - Math.random() * w * 0.4, y: Math.random() * h, size: Math.random() * 3 + 2, speed: Math.random() * 2 + 1 }); }
		function resize() { [h5c1, h5c2].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h5c1.width; h = h5c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.016; x1.fillStyle = 'rgba(0,0,0,0.06)'; x1.fillRect(0, 0, w, h);
			const mid = w / 2 + (mx / w - 0.5) * 100; x2.clearRect(0, 0, w, h); x2.strokeStyle = 'rgba(255,255,255,0.05)'; x2.setLineDash([4, 8]); x2.lineWidth = 1; x2.beginPath(); x2.moveTo(mid, 0); x2.lineTo(mid, h); x2.stroke(); x2.setLineDash([]);
			bids.forEach(b => { b.x += b.vx; b.y += b.vy; if (b.x > mid - 10) { b.vx *= -0.5; b.x = mid - 10 - Math.random() * 20; sparks.push({ x: mid, y: b.y, vx: (Math.random() - 0.5) * 6, vy: (Math.random() - 0.5) * 6, life: 1, color: '255,255,200' }); } if (b.x < -20) { b.x = -20; b.vx = 1 + Math.random() * 2; } if (b.y < 0 || b.y > h) b.vy *= -1; x1.beginPath(); x1.arc(b.x, b.y, b.size, 0, Math.PI * 2); x1.fillStyle = `rgba(0,255,136,${0.4 + Math.sin(t * 5 + b.y * 0.01) * 0.1})`; x1.fill(); });
			asks.forEach(a => { a.x += a.vx; a.y += a.vy; if (a.x < mid + 10) { a.vx *= -0.5; a.x = mid + 10 + Math.random() * 20; sparks.push({ x: mid, y: a.y, vx: (Math.random() - 0.5) * 6, vy: (Math.random() - 0.5) * 6, life: 1, color: '255,255,200' }); } if (a.x > w + 20) { a.x = w + 20; a.vx = -(1 + Math.random() * 2); } if (a.y < 0 || a.y > h) a.vy *= -1; x1.beginPath(); x1.arc(a.x, a.y, a.size, 0, Math.PI * 2); x1.fillStyle = `rgba(255,51,68,${0.4 + Math.sin(t * 5 + a.y * 0.01) * 0.1})`; x1.fill(); });
			sparks.forEach(s => { s.x += s.vx; s.y += s.vy; s.vx *= 0.96; s.vy *= 0.96; s.life -= 0.03; x2.beginPath(); x2.arc(s.x, s.y, 2 * s.life, 0, Math.PI * 2); x2.fillStyle = `rgba(${s.color},${s.life})`; x2.shadowBlur = 6; x2.shadowColor = `rgba(${s.color},0.5)`; x2.fill(); x2.shadowBlur = 0; }); sparks.filter(s => s.life > 0);
			const clashGrd = x2.createRadialGradient(mid, h / 2, 0, mid, h / 2, 100); clashGrd.addColorStop(0, `rgba(255,255,200,${0.02 + Math.sin(t * 8) * 0.01})`); clashGrd.addColorStop(1, 'transparent'); x2.fillStyle = clashGrd; x2.fillRect(mid - 100, h / 2 - 100, 200, 200);
			x2.font = '9px "JetBrains Mono"'; x2.fillStyle = 'rgba(0,255,136,0.15)'; x2.fillText('BIDS', 30, 30); x2.fillStyle = 'rgba(255,51,68,0.15)'; x2.textAlign = 'right'; x2.fillText('ASKS', w - 30, 30); x2.textAlign = 'start';
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h5', start: 'top 70%', onEnter: () => { active = true; const tl = gsap.timeline({ delay: 0.3 }); tl.to('#h5t', { opacity: 1, duration: 0.5 }, 1); tl.add(() => shake(document.getElementById('h5-content')!, 15, 0.3), 1); tl.to('#h5s', { opacity: 1, duration: 1 }, 1.5); } }); triggers.push(trigger);
	}

	function initGamma() {
		const x1 = h6c1.getContext('2d')!, x2 = h6c2.getContext('2d')!; let w = h6c1.width = h6c1.parentElement!.offsetWidth, h = h6c1.height = h6c1.parentElement!.offsetHeight;
		let t = 0, active = false, rampY = 0; const chain: { strike: number; x: number; y: number; gamma: number; lit: number; particles: { x: number; y: number; vx: number; vy: number; life: number; size: number }[] }[] = [];
		for (let i = 0; i < 25; i++) chain.push({ strike: 5800 + i * 5, x: w * 0.15 + i * (w * 0.7 / 25), y: h * 0.5, gamma: Math.exp(-Math.pow(i - 12, 2) / 20) * 0.8, lit: 0, particles: [] });
		function detonateStrike(idx: number) { if (idx >= chain.length || idx < 0) return; const s = chain[idx]; if (s.lit > 0.5) return; gsap.to(s, { lit: 1, duration: 0.3, ease: 'power4.out' }); for (let i = 0; i < 30; i++) { const a = Math.random() * Math.PI * 2, spd = Math.random() * 5 + 2; s.particles.push({ x: s.x, y: s.y, vx: Math.cos(a) * spd, vy: Math.sin(a) * spd, life: 1, size: Math.random() * 3 + 1 }); } timeouts.push(window.setTimeout(() => detonateStrike(idx + 1), 80)); timeouts.push(window.setTimeout(() => detonateStrike(idx - 1), 120)); }
		function resize() { [h6c1, h6c2].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h6c1.width; h = h6c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; x1.fillStyle = 'rgba(10,0,5,0.06)'; x1.fillRect(0, 0, w, h); x2.clearRect(0, 0, w, h);
			x1.beginPath(); chain.forEach((s, i) => { const gy = h * 0.5 - s.gamma * h * 0.3 * (1 + s.lit * 0.5); if (i === 0) x1.moveTo(s.x, gy); else x1.lineTo(s.x, gy); }); x1.strokeStyle = 'rgba(255,0,200,0.15)'; x1.lineWidth = 2; x1.shadowBlur = 10; x1.shadowColor = 'rgba(255,0,200,0.3)'; x1.stroke(); x1.shadowBlur = 0;
			chain.forEach(s => { const barH = s.gamma * h * 0.25 * (1 + s.lit * 2), glow = s.lit; x2.fillStyle = `rgba(255,${Math.floor(glow * 100)},${Math.floor(200 - glow * 100)},${0.1 + glow * 0.4})`; x2.fillRect(s.x - 3, h * 0.5 - barH, 6, barH); if (glow > 0.3) { x2.shadowBlur = 20; x2.shadowColor = `rgba(255,0,200,${glow * 0.5})`; x2.fillRect(s.x - 3, h * 0.5 - barH, 6, barH); x2.shadowBlur = 0; } s.particles.forEach(p => { p.x += p.vx; p.y += p.vy; p.vy += 0.05; p.life -= 0.02; if (p.life > 0) { x2.beginPath(); x2.arc(p.x, p.y, p.size * p.life, 0, Math.PI * 2); x2.fillStyle = `rgba(255,100,200,${p.life * 0.6})`; x2.fill(); } }); s.particles = s.particles.filter(p => p.life > 0); });
			if (rampY > 0) { const rampPath: { x: number; y: number }[] = []; for (let i = 0; i < w; i += 5) { const prog = i / w; const ry = h * 0.8 - Math.pow(prog, 3) * rampY * h * 0.6; rampPath.push({ x: i, y: ry }); } x2.beginPath(); rampPath.forEach((p, i) => { if (i === 0) x2.moveTo(p.x, p.y); else x2.lineTo(p.x, p.y); }); x2.strokeStyle = 'rgba(255,100,0,0.3)'; x2.lineWidth = 2; x2.stroke(); }
			const greeks = ['Γ', 'Δ', 'θ', 'Σ', 'Ω']; greeks.forEach((g, i) => { const gx = w * 0.2 + i * w * 0.15 + Math.sin(t + i) * 20; const gy = h * 0.15 + Math.cos(t * 0.7 + i) * 15; x2.fillStyle = `rgba(255,0,200,${0.06 + Math.sin(t * 2 + i) * 0.02})`; x2.font = 'bold 40px serif'; x2.fillText(g, gx, gy); });
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h6', start: 'top 70%', onEnter: () => { active = true; timeouts.push(window.setTimeout(() => detonateStrike(12), 1000)); gsap.to({ v: 0 }, { v: 1, duration: 3, delay: 1.5, ease: 'power3.in', onUpdate: function() { rampY = this.targets()[0].v; } }); const tl = gsap.timeline({ delay: 0.3 }); tl.to('#h6f', { opacity: 0.3, duration: 0.05 }, 2); tl.to('#h6f', { opacity: 0, duration: 0.8 }, 2.1); tl.to('#h6t', { opacity: 1, duration: 0.5 }, 2); tl.add(() => shake(document.getElementById('h6-content')!, 25, 0.5), 2); tl.to('#h6s', { opacity: 1, duration: 0.8 }, 2.5); tl.to('#h6s2', { opacity: 1, duration: 0.8 }, 3); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initMomentum(); initSpread(); initGamma(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="h4" style="background:#050200">
	<canvas bind:this={h4c1}></canvas>
	<canvas bind:this={h4c2} class="c2"></canvas>
	<div class="vig h4-vig"></div>
	<div class="grn"></div>
	<div class="flash" id="h4f" style="background:#fff"></div>
	<div class="ct" id="h4-content">
		<h1 class="t1" id="h4t">BREAKOUT</h1>
		<p class="t2" id="h4s">RESISTANCE IS TEMPORARY. MOMENTUM IS FOREVER.</p>
	</div>
	<span class="lbl h4-lbl">04 — MOMENTUM</span>
	<span class="idx h4-idx">04</span>
</section>
<div class="sp"></div>

<section class="hero" id="h5" style="background:#000">
	<canvas bind:this={h5c1}></canvas>
	<canvas bind:this={h5c2} class="c2"></canvas>
	<div class="vig h5-vig"></div>
	<div class="grn"></div>
	<div class="flash" id="h5f" style="background:rgba(255,255,255,0.8)"></div>
	<div class="ct" id="h5-content">
		<h1 class="t1" id="h5t">THE SPREAD</h1>
		<p class="t2" id="h5s">WHERE BUYERS AND SELLERS GO TO WAR</p>
	</div>
	<span class="lbl h5-lbl">05 — THE SPREAD</span>
	<span class="idx h5-idx">05</span>
</section>
<div class="sp"></div>

<section class="hero" id="h6" style="background:#0a0005">
	<canvas bind:this={h6c1}></canvas>
	<canvas bind:this={h6c2} class="c2"></canvas>
	<div class="vig h6-vig"></div>
	<div class="grn"></div>
	<div class="flash" id="h6f" style="background:rgba(255,0,200,0.6)"></div>
	<div class="ct" id="h6-content">
		<h1 class="t1" id="h6t">GAMMA</h1>
		<p class="t2" id="h6s">THE CHAIN REACTION THEY CAN'T STOP</p>
		<p class="t3" id="h6s2">[ Γ EXPOSURE CRITICAL ]</p>
	</div>
	<span class="lbl h6-lbl">06 — GAMMA SQUEEZE</span>
	<span class="idx h6-idx">06</span>
</section>

<style>
	:global(*) { margin: 0; padding: 0; box-sizing: border-box; }
	:global(html) { overflow-x: hidden; background: #000; }
	.hero { position: relative; width: 100%; height: 100vh; overflow: hidden; display: flex; align-items: center; justify-content: center; color: #fff; font-family: 'JetBrains Mono', monospace; }
	.hero canvas { position: absolute; inset: 0; width: 100%; height: 100%; z-index: 1; }
	.c2 { z-index: 2; }
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
	.h4-vig { background: radial-gradient(ellipse, transparent 30%, rgba(5,2,0,0.85) 100%); }
	.h5-vig { background: radial-gradient(ellipse, transparent 35%, rgba(0,0,0,0.8) 100%); }
	.h6-vig { background: radial-gradient(ellipse, transparent 30%, rgba(10,0,5,0.85) 100%); }
	#h4t { background: linear-gradient(180deg, #00ff88, #00cc66, #006633); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#h4s { color: rgba(0,255,136,0.5); }
	#h5t { color: #fff; text-shadow: 0 0 60px rgba(255,255,255,0.2); }
	#h5s { color: rgba(255,255,255,0.3); }
	#h6t { background: linear-gradient(90deg, #ff00cc, #ff6600); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#h6s { color: rgba(255,0,200,0.5); }
	#h6s2 { color: rgba(255,100,0,0.3); }
	.h4-lbl, .h4-idx { color: #00ff88; }
	.h5-lbl { color: #888; }
	.h6-lbl, .h6-idx { color: #ff00cc; }
</style>
