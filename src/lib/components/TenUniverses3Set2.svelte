<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let c4: HTMLCanvasElement, c5: HTMLCanvasElement, c6: HTMLCanvasElement;
	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }

	function shake(el: HTMLElement, intensity = 20, dur = 0.5) {
		const s = Math.floor(dur / 0.025);
		for (let i = 0; i < s; i++) {
			const d = 1 - i / s;
			gsap.set(el, { x: (Math.random() - 0.5) * intensity * d, y: (Math.random() - 0.5) * intensity * d, delay: i * 0.025 });
		}
		gsap.set(el, { x: 0, y: 0, delay: s * 0.025 });
	}

	function initStorm() {
		const x = c4.getContext('2d')!; let w = c4.width = c4.parentElement!.offsetWidth, h = c4.height = c4.parentElement!.offsetHeight;
		const bits: { angle: number; dist: number; speed: number; driftOut: number; char: string; size: number; alpha: number; z: number }[] = [], arcs: { points: { a: number; r: number }[]; life: number }[] = [];
		const CHARS = '01'; let t = 0, active = false;
		for (let i = 0; i < 600; i++) bits.push({ angle: Math.random() * Math.PI * 2, dist: Math.random() * Math.min(w, h) * 0.5, speed: 0.002 + Math.random() * 0.008, driftOut: Math.random() * 0.3 - 0.15, char: CHARS[Math.floor(Math.random() * 2)], size: Math.random() * 10 + 8, alpha: Math.random() * 0.6 + 0.1, z: Math.random() });
		function spawnArc() { if (!active) return; const startAngle = Math.random() * Math.PI * 2, points: { a: number; r: number }[] = []; let a = startAngle, r = 50; for (let i = 0; i < 15; i++) { a += Math.random() * 0.4 - 0.2; r += Math.random() * 30 + 10; points.push({ a, r }); } arcs.push({ points, life: 1 }); timeouts.push(window.setTimeout(spawnArc, Math.random() * 500 + 100)); }
		function resize() { w = c4.width = c4.parentElement!.offsetWidth; h = c4.height = c4.parentElement!.offsetHeight; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.01; x.fillStyle = 'rgba(3,3,3,0.12)'; x.fillRect(0, 0, w, h);
			const cx = w / 2 + (mx / w - 0.5) * 30, cy = h / 2 + (my / h - 0.5) * 30;
			x.font = '10px "JetBrains Mono"'; bits.forEach(b => { b.angle += b.speed * (1 + b.z); b.dist += b.driftOut; if (b.dist > Math.max(w, h) * 0.7) b.dist = 20 + Math.random() * 50; if (b.dist < 10) b.dist = Math.min(w, h) * 0.5; if (Math.random() > 0.99) b.char = CHARS[Math.floor(Math.random() * 2)]; const px = cx + Math.cos(b.angle) * b.dist, py = cy + Math.sin(b.angle) * b.dist, distFade = Math.min(1, b.dist / 200); x.fillStyle = `rgba(255,255,255,${b.alpha * distFade * (0.5 + b.z * 0.5)})`; x.fillText(b.char, px, py); });
			arcs.forEach(arc => { arc.life *= 0.95; x.beginPath(); arc.points.forEach((p, i) => { const jx = cx + Math.cos(p.a + t * 2) * (p.r + (Math.random() - 0.5) * 10), jy = cy + Math.sin(p.a + t * 2) * (p.r + (Math.random() - 0.5) * 10); if (i === 0) x.moveTo(jx, jy); else x.lineTo(jx, jy); }); x.strokeStyle = `rgba(255,255,255,${arc.life * 0.6})`; x.lineWidth = 1 + arc.life; x.shadowBlur = 15; x.shadowColor = `rgba(200,200,255,${arc.life * 0.5})`; x.stroke(); x.shadowBlur = 0; }); arcs.filter(a => a.life > 0.05);
			const eyeR = 30 + Math.sin(t * 3) * 5; x.beginPath(); x.arc(cx, cy, eyeR, 0, Math.PI * 2); x.fillStyle = `rgba(255,255,255,${0.03 + Math.sin(t * 4) * 0.02})`; x.fill(); x.strokeStyle = 'rgba(255,255,255,0.08)'; x.lineWidth = 1; x.stroke();
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s4', start: 'top 80%', onEnter: () => { active = true; spawnArc(); const tl = gsap.timeline({ delay: 0.3 }); tl.to('#s4t', { opacity: 1, duration: 0.1, ease: 'none' }, 0.5); tl.add(() => shake(document.getElementById('s4-content')!, 30, 0.4), 0.5); tl.to('#s4s', { opacity: 1, duration: 1 }, 1.2); } }); triggers.push(trigger);
	}

	function initHeartbeat() {
		const x = c5.getContext('2d')!; let w = c5.width = c5.parentElement!.offsetWidth, h = c5.height = c5.parentElement!.offsetHeight;
		let t = 0, active = false; const pulseParticles: { x: number; y: number; vx: number; vy: number; life: number; size: number }[] = [];
		function ekgValue(phase: number) { const p = phase % 1; if (p < 0.1) return 0; if (p < 0.15) return Math.sin((p - 0.1) / 0.05 * Math.PI) * 0.3; if (p < 0.2) return 0; if (p < 0.22) return -0.15; if (p < 0.26) return 1; if (p < 0.3) return -0.3; if (p < 0.35) return 0; if (p < 0.45) return Math.sin((p - 0.35) / 0.1 * Math.PI) * 0.2; return 0; }
		function pulseBurst() { if (!active) return; for (let i = 0; i < 30; i++) { const angle = Math.random() * Math.PI * 2, speed = Math.random() * 4 + 2; pulseParticles.push({ x: w / 2, y: h / 2, vx: Math.cos(angle) * speed, vy: Math.sin(angle) * speed, life: 1, size: Math.random() * 3 + 1 }); } gsap.to('#s5', { boxShadow: 'inset 0 0 120px rgba(255,0,50,0.15)', duration: 0.15, onComplete: () => gsap.to('#s5', { boxShadow: 'inset 0 0 120px rgba(255,0,50,0)', duration: 0.8 }) }); timeouts.push(window.setTimeout(pulseBurst, 800 + Math.random() * 200)); }
		function resize() { w = c5.width = c5.parentElement!.offsetWidth; h = c5.height = c5.parentElement!.offsetHeight; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.012; x.fillStyle = 'rgba(5,0,0,0.08)'; x.fillRect(0, 0, w, h);
			x.beginPath(); const ekgY = h * 0.5; for (let i = 0; i < w; i += 2) { const phase = ((i / w) * 2 + t) % 2, val = ekgValue(phase % 1) * 150, px = i, py = ekgY - val; if (i === 0) x.moveTo(px, py); else x.lineTo(px, py); } x.strokeStyle = 'rgba(255,0,50,0.4)'; x.lineWidth = 2; x.shadowBlur = 20; x.shadowColor = 'rgba(255,0,50,0.5)'; x.stroke(); x.shadowBlur = 0;
			for (const offset of [-50, 50, -100, 100]) { x.beginPath(); for (let i = 0; i < w; i += 3) { const phase = ((i / w) * 2 + t + offset * 0.001) % 2, val = ekgValue(phase % 1) * 100; if (i === 0) x.moveTo(i, ekgY + offset - val); else x.lineTo(i, ekgY + offset - val); } x.strokeStyle = 'rgba(255,0,50,0.06)'; x.lineWidth = 1; x.stroke(); }
			const pulsePhase = (t * 1.2) % 1, pulseR = pulsePhase * Math.max(w, h) * 0.5; x.beginPath(); x.arc(w / 2, h / 2, pulseR, 0, Math.PI * 2); x.strokeStyle = `rgba(255,0,50,${(1 - pulsePhase) * 0.1})`; x.lineWidth = 2; x.stroke();
			pulseParticles.forEach(p => { p.x += p.vx; p.y += p.vy; p.vx *= 0.98; p.vy *= 0.98; p.life -= 0.015; x.beginPath(); x.arc(p.x, p.y, p.size * p.life, 0, Math.PI * 2); x.fillStyle = `rgba(255,30,60,${p.life * 0.6})`; x.shadowBlur = 8; x.shadowColor = 'rgba(255,0,50,0.4)'; x.fill(); x.shadowBlur = 0; }); pulseParticles.filter(p => p.life > 0);
			const bpm = Math.floor(72 + Math.sin(t * 0.5) * 8); x.fillStyle = 'rgba(255,0,50,0.15)'; x.font = 'bold 80px Orbitron'; x.textAlign = 'right'; x.fillText(bpm.toString(), w - 40, h - 40); x.font = '12px "JetBrains Mono"'; x.fillText('BPM', w - 40, h - 15); x.textAlign = 'start';
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s5', start: 'top 80%', onEnter: () => { active = true; pulseBurst(); const tl = gsap.timeline({ delay: 0.3 }); tl.to('#s5t', { opacity: 1, duration: 0.15, ease: 'power4.out' }, 0.8); tl.add(() => shake(document.getElementById('s5-content')!, 20, 0.3), 0.8); tl.to('#s5s', { opacity: 1, duration: 1 }, 1.5); } }); triggers.push(trigger);
	}

	function initBlueprint() {
		const x = c6.getContext('2d')!; let w = c6.width = c6.parentElement!.offsetWidth, h = c6.height = c6.parentElement!.offsetHeight;
		let t = 0, active = false; const shapes: { x: number; y: number; targetX: number; targetY: number; size: number; type: 'cube' | 'pyramid' | 'sphere'; progress: number }[] = [], grid: { x: number; y: number; alpha: number }[] = [];
		for (let i = 0; i < 12; i++) { const angle = (i / 12) * Math.PI * 2; shapes.push({ x: w / 2 + Math.cos(angle) * 300, y: h / 2 + Math.sin(angle) * 200, targetX: w / 2 + Math.cos(angle) * 150, targetY: h / 2 + Math.sin(angle) * 100, size: 30 + Math.random() * 40, type: ['cube', 'pyramid', 'sphere'][Math.floor(Math.random() * 3)] as 'cube' | 'pyramid' | 'sphere', progress: 0 }); }
		for (let gx = 0; gx < w; gx += 40) for (let gy = 0; gy < h; gy += 40) if (Math.random() > 0.7) grid.push({ x: gx, y: gy, alpha: Math.random() * 0.1 });
		function resize() { w = c6.width = c6.parentElement!.offsetWidth; h = c6.height = c6.parentElement!.offsetHeight; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; x.fillStyle = 'rgba(0,8,18,0.06)'; x.fillRect(0, 0, w, h);
			const mouseOffsetX = (mx / w - 0.5) * 20, mouseOffsetY = (my / h - 0.5) * 20;
			grid.forEach(g => { x.fillStyle = `rgba(68,153,255,${g.alpha + Math.sin(t + g.x * 0.01) * 0.05})`; x.fillRect(g.x, g.y, 1, 1); });
			for (let i = 0; i <= 10; i++) { const y = h * 0.3 + i * h * 0.04; x.beginPath(); x.moveTo(0, y); for (let px = 0; px < w; px += 10) x.lineTo(px, y + Math.sin(px * 0.02 + t + i) * 2); x.strokeStyle = `rgba(68,153,255,${0.03 + i * 0.005})`; x.lineWidth = 0.5; x.stroke(); }
			shapes.forEach((s, i) => { const currX = s.x + (s.targetX - s.x) * s.progress + mouseOffsetX * (i % 3 - 1), currY = s.y + (s.targetY - s.y) * s.progress + mouseOffsetY * (i % 3 - 1), p = 0.3 + s.progress * 0.7; x.save(); x.translate(currX, currY); x.rotate(t * 0.2 + i); x.strokeStyle = `rgba(100,180,255,${p * 0.6})`; x.lineWidth = 1.5; if (s.type === 'cube') { const r = s.size * p; x.strokeRect(-r, -r, r * 2, r * 2); x.beginPath(); x.moveTo(-r, -r); x.lineTo(-r * 0.87, -r * 0.5); x.lineTo(r * 0.87, -r * 0.5); x.lineTo(r, -r); x.moveTo(-r, r); x.lineTo(-r * 0.87, r * 0.5); x.lineTo(r * 0.87, r * 0.5); x.lineTo(r, r); x.moveTo(-r * 0.87, -r * 0.5); x.lineTo(-r * 0.87, r * 0.5); x.moveTo(r * 0.87, -r * 0.5); x.lineTo(r * 0.87, r * 0.5); x.stroke(); } else if (s.type === 'pyramid') { const r = s.size * p; x.beginPath(); x.moveTo(0, -r); for (let j = 1; j < 6; j++) { const a = (j / 6) * Math.PI * 2 - Math.PI / 2; x.lineTo(Math.cos(a) * r, Math.sin(a) * r); } x.closePath(); x.stroke(); x.beginPath(); x.moveTo(0, -r * 0.3); for (let j = 0; j < 6; j++) { const a = (j / 6) * Math.PI * 2 - Math.PI / 2 + Math.PI / 6; x.lineTo(Math.cos(a) * r * 0.5, Math.sin(a) * r * 0.5); } x.closePath(); x.stroke(); } else { x.beginPath(); x.arc(0, 0, s.size * p * 0.5, 0, Math.PI * 2); x.stroke(); x.beginPath(); for (let j = 0; j < 3; j++) { const a = (j / 3) * Math.PI * 2; x.moveTo(0, 0); x.lineTo(Math.cos(a) * s.size * p * 0.5, Math.sin(a) * s.size * p * 0.5); } x.stroke(); } x.fillStyle = `rgba(68,153,255,${0.5 * p})`; x.beginPath(); x.arc(0, 0, 2, 0, Math.PI * 2); x.fill(); x.restore(); });
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s6', start: 'top 80%', onEnter: () => { active = true; shapes.forEach((s, i) => { gsap.to(s, { progress: 1, duration: 1.5, delay: i * 0.15, ease: 'power2.out' }); }); const tl = gsap.timeline({ delay: 1 }); tl.to('#s6t', { opacity: 1, duration: 0.8 }, 0.5); tl.to('#s6s', { opacity: 1, duration: 1 }, 1.2); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initStorm(); initHeartbeat(); initBlueprint(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero h-storm" id="s4">
	<canvas bind:this={c4}></canvas>
	<div class="vignette s4-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s4-content">
		<h1 class="t1" id="s4t">OVERRIDE</h1>
		<p class="t2" id="s4s">CHAOS IS THE SYSTEM RECALIBRATING</p>
	</div>
	<span class="label">04 — STORM</span>
	<span class="num">04</span>
</section>
<div class="divider"></div>

<section class="hero h-heartbeat" id="s5">
	<canvas bind:this={c5}></canvas>
	<div class="vignette s5-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s5-content">
		<h1 class="t1" id="s5t">PULSE</h1>
		<p class="t2" id="s5s">THE MARKET HAS A HEARTBEAT</p>
	</div>
	<span class="label">05 — PULSE</span>
	<span class="num">05</span>
</section>
<div class="divider"></div>

<section class="hero h-blueprint" id="s6">
	<canvas bind:this={c6}></canvas>
	<div class="vignette s6-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s6-content">
		<h1 class="t1" id="s6t">ARCHITECT</h1>
		<p class="t2" id="s6s">ENGINEERED FOR PRECISION</p>
	</div>
	<span class="label">06 — BLUEPRINT</span>
	<span class="num">06</span>
</section>

<style>
	:global(*) { margin: 0; padding: 0; box-sizing: border-box; }
	:global(html) { overflow-x: hidden; }
	.hero { position: relative; height: 100vh; overflow: hidden; display: flex; align-items: center; justify-content: center; background: #000; color: #fff; font-family: 'JetBrains Mono', monospace; }
	.hero canvas { position: absolute; inset: 0; z-index: 1; width: 100%; height: 100%; }
	.hero .content { position: relative; z-index: 10; text-align: center; pointer-events: none; }
	.hero .grain { position: absolute; inset: 0; z-index: 50; pointer-events: none; opacity: 0.04; background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 512 512' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.8' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E"); background-size: 256px; }
	.hero .vignette { position: absolute; inset: 0; z-index: 40; pointer-events: none; }
	.hero .label { position: absolute; top: 25px; left: 30px; z-index: 60; font-size: 0.5rem; letter-spacing: 0.5em; text-transform: uppercase; opacity: 0.2; }
	.hero .num { position: absolute; bottom: 30px; right: 35px; z-index: 60; font-family: 'Bebas Neue', sans-serif; font-size: 6rem; line-height: 1; opacity: 0.03; }
	.h-storm { background: #030303; }
	.h-heartbeat { background: #050000; }
	.h-blueprint { background: #000812; }
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(4rem, 14vw, 12rem); line-height: 0.85; letter-spacing: 0.02em; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.5rem, 1.2vw, 0.75rem); letter-spacing: 0.6em; margin-top: 20px; opacity: 0; }
	.divider { height: 5vh; background: #000; }
	.s4-vignette { background: radial-gradient(ellipse, transparent 35%, rgba(3,3,3,0.8) 100%); }
	.s5-vignette { background: radial-gradient(ellipse, transparent 35%, rgba(5,0,0,0.8) 100%); }
	.s6-vignette { background: radial-gradient(ellipse, transparent 40%, rgba(0,8,18,0.7) 100%); }
	#s4t { color: #fff; text-shadow: 0 0 40px rgba(255,255,255,0.3); }
	#s4s { color: rgba(255,255,255,0.4); }
	#s5t { color: #ff0033; text-shadow: 0 0 80px rgba(255,0,50,0.4); }
	#s5s { color: rgba(255,0,50,0.4); }
	#s6t { color: #4499ff; text-shadow: 0 0 60px rgba(68,153,255,0.3); }
	#s6s { color: rgba(68,153,255,0.4); }
	.h-storm .label { color: #888; }
	.h-heartbeat .label { color: #ff0033; }
	.h-heartbeat .num { color: #ff0033; }
	.h-blueprint .label { color: #4499ff; }
	.h-blueprint .num { color: #4499ff; }
</style>
