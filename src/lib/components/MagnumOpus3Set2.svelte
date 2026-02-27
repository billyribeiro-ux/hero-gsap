<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let s4a: HTMLCanvasElement, s4b: HTMLCanvasElement;
	let s5a: HTMLCanvasElement, s5b: HTMLCanvasElement;
	let s6a: HTMLCanvasElement, s6b: HTMLCanvasElement, s6c: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }
	function shake(el: HTMLElement, intensity = 20, dur = 0.5) { const s = Math.floor(dur / 0.016); for (let i = 0; i < s; i++) { const f = Math.pow(1 - i / s, 2); gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.6, delay: i * 0.016 }); } gsap.set(el, { x: 0, y: 0, delay: s * 0.016 }); }

	function initForge() {
		const cv = [s4a, s4b];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s4a.width = s4a.parentElement!.offsetWidth, h = s4a.height = s4a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const sparks: { x: number; y: number; vx: number; vy: number; size: number; life: number; hue: number }[] = [];
		const embers: { x: number; y: number; vx: number; vy: number; size: number; life: number; hue: number }[] = [];
		for (let i = 0; i < 200; i++) embers.push({ x: Math.random() * 2000, y: 1200 + Math.random() * 100, vx: (Math.random() - 0.5) * 1, vy: -Math.random() * 2 - 0.5, size: Math.random() * 3 + 0.5, life: Math.random(), hue: 10 + Math.random() * 40 });
		const anvilX = 0.5, anvilY = 0.7;
		function hammerStrike() {
			if (!on) return; const ax = w * anvilX, ay = h * anvilY;
			for (let i = 0; i < 120; i++) { const a = -Math.PI * 0.9 + Math.random() * Math.PI * 0.8, spd = 5 + Math.random() * 15; sparks.push({ x: ax + (Math.random() - 0.5) * 30, y: ay, vx: Math.cos(a) * spd * (Math.random() > 0.5 ? 1 : -1), vy: Math.sin(a) * spd, size: Math.random() * 3 + 0.5, life: 1, hue: 20 + Math.random() * 40 }); }
			shake(document.getElementById('s4-content')!, 12, 0.2); gsap.to('#s4f', { opacity: 0.3, duration: 0.04, onComplete: () => gsap.to('#s4f', { opacity: 0, duration: 0.4 }) }); timeouts.push(window.setTimeout(hammerStrike, 400 + Math.random() * 300));
		}
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s4a.width; h = s4a.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; const [xa, xb] = ctx; const ax = w * anvilX, ay = h * anvilY;
			xa.fillStyle = 'rgba(0,0,0,0.08)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h);
			xa.globalCompositeOperation = 'lighter'; const heatGrd = xa.createRadialGradient(ax, ay, 0, ax, ay, 150); heatGrd.addColorStop(0, 'rgba(255,100,0,0.06)'); heatGrd.addColorStop(1, 'transparent'); xa.fillStyle = heatGrd; xa.fillRect(ax - 150, ay - 150, 300, 300); xa.globalCompositeOperation = 'source-over';
			xb.beginPath(); xb.ellipse(ax, ay, 40 + Math.sin(t * 3) * 5, 8, 0, 0, Math.PI * 2); xb.fillStyle = `rgba(255,150,30,${0.5 + Math.sin(t * 5) * 0.2})`; xb.shadowBlur = 30; xb.shadowColor = 'rgba(255,100,0,0.6)'; xb.fill(); xb.beginPath(); xb.ellipse(ax, ay, 20, 4, 0, 0, Math.PI * 2); xb.fillStyle = `rgba(255,230,150,${0.6 + Math.sin(t * 8) * 0.2})`; xb.fill(); xb.shadowBlur = 0;
			xa.globalCompositeOperation = 'lighter'; sparks.forEach(s => { s.vy += 0.25; s.x += s.vx; s.y += s.vy; s.life -= 0.015; s.vx *= 0.995; if (s.life > 0) { xa.beginPath(); xa.arc(s.x, s.y, s.size * s.life, 0, Math.PI * 2); xa.fillStyle = `hsla(${s.hue},100%,${50 + s.life * 30}%,${s.life * 0.9})`; xa.fill(); xa.beginPath(); xa.moveTo(s.x, s.y); xa.lineTo(s.x - s.vx * 2, s.y - s.vy * 2); xa.strokeStyle = `hsla(${s.hue},100%,60%,${s.life * 0.5})`; xa.lineWidth = s.size * 0.6; xa.stroke(); } }); sparks.filter(s => s.life > 0); xa.globalCompositeOperation = 'source-over';
			embers.forEach(e => { e.x += e.vx + Math.sin(t + e.x * 0.002) * 0.5; e.y += e.vy; e.life -= 0.002; if (e.life <= 0) { e.y = h + 20; e.x = Math.random() * w; e.life = 1; } const fl = 0.3 + Math.sin(t * 18 + e.x) * 0.7; xb.beginPath(); xb.arc(e.x, e.y, e.size * e.life, 0, Math.PI * 2); xb.fillStyle = `hsla(${e.hue},100%,55%,${e.life * fl * 0.5})`; xb.shadowBlur = e.size * 4; xb.shadowColor = `hsla(${e.hue},100%,50%,${e.life * 0.3})`; xb.fill(); xb.shadowBlur = 0; });
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s4', start: 'top 60%', onEnter: () => { on = true; hammerStrike(); const tl = gsap.timeline({ delay: 1.5 }); tl.to('#s4t', { opacity: 1, duration: 0.4 }, 0); tl.from('#s4t', { scale: 1.5, filter: 'blur(8px)', duration: 0.4, ease: 'power4.out' }, 0); tl.to('#s4u', { opacity: 1, duration: 1 }, 0.6); tl.to('#s4l', { opacity: 0.12, duration: 0.5 }, 0.3); } }); triggers.push(trigger);
	}

	function initConvergence() {
		const cv = [s5a, s5b];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s5a.width = s5a.parentElement!.offsetWidth, h = s5a.height = s5a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const lines: { sx: number; sy: number; progress: number; width: number; hue: number; speed: number }[] = [];
		function initLines() { lines.length = 0; const CX = w / 2, CY = h / 2; for (let i = 0; i < 60; i++) { const edge = Math.floor(Math.random() * 4); let sx, sy; if (edge === 0) { sx = Math.random() * w; sy = -50; } else if (edge === 1) { sx = w + 50; sy = Math.random() * h; } else if (edge === 2) { sx = Math.random() * w; sy = h + 50; } else { sx = -50; sy = Math.random() * h; } lines.push({ sx, sy, progress: 0, width: Math.random() * 2 + 1, hue: 150 + Math.random() * 60, speed: 0 }); } }
		initLines();
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s5a.width; h = s5a.height; initLines(); }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; const [xa, xb] = ctx; const CX = w / 2, CY = h / 2;
			xa.fillStyle = 'rgba(0,0,0,0.06)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h);
			xa.globalCompositeOperation = 'lighter'; lines.forEach(l => { if (l.progress >= 1) return; const p = l.progress, headX = l.sx + (CX - l.sx) * p, headY = l.sy + (CY - l.sy) * p, tailP = Math.max(0, p - 0.3), tailX = l.sx + (CX - l.sx) * tailP, tailY = l.sy + (CY - l.sy) * tailP; xa.beginPath(); xa.moveTo(tailX, tailY); xa.lineTo(headX, headY); xa.strokeStyle = `hsla(${l.hue},100%,60%,${0.6 * Math.min(1, p * 3)})`; xa.lineWidth = l.width; xa.stroke(); }); xa.globalCompositeOperation = 'source-over';
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s5', start: 'top 60%', onEnter: () => { on = true; lines.forEach((l, i) => { gsap.to(l, { progress: 1, duration: 2 + Math.random(), delay: i * 0.03, ease: 'power2.in' }); }); const tl = gsap.timeline(); tl.to('#s5f', { opacity: 1, duration: 0.05 }, 1.5); tl.to('#s5f', { opacity: 0, duration: 1 }, 1.55); tl.add(() => shake(document.getElementById('s5-content')!, 30, 0.6), 1.5); tl.to('#s5t', { opacity: 1, duration: 0.5 }, 1.8); tl.from('#s5t', { scale: 3, filter: 'blur(20px)', duration: 0.5, ease: 'power4.out' }, 1.8); tl.to('#s5u', { opacity: 1, duration: 1 }, 2.5); tl.to('#s5l', { opacity: 0.12, duration: 0.5 }, 2.2); } }); triggers.push(trigger);
	}

	function initBloodbath() {
		const cv = [s6a, s6b, s6c];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s6a.width = s6a.parentElement!.offsetWidth, h = s6a.height = s6a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const rain: { x: number; y: number; vy: number; len: number; width: number; alpha: number }[] = [];
		for (let i = 0; i < 400; i++) rain.push({ x: Math.random() * w, y: Math.random() * h, vy: 8 + Math.random() * 12, len: 15 + Math.random() * 25, width: 1 + Math.random() * 2, alpha: 0.3 + Math.random() * 0.5 });
		const splashes: { x: number; y: number; vx: number; vy: number; size: number; life: number }[] = [];
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s6a.width; h = s6a.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.016; const [xa, xb, xc] = ctx;
			xa.fillStyle = 'rgba(20,0,0,0.15)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xc.clearRect(0, 0, w, h);
			xa.strokeStyle = 'rgba(255,30,30,0.6)'; xa.lineWidth = 2; xa.lineCap = 'round'; rain.forEach(r => { r.y += r.vy; if (r.y > h) { r.y = -r.len; r.x = Math.random() * w; if (Math.random() > 0.7) for (let i = 0; i < 3; i++) splashes.push({ x: r.x, y: h - 5, vx: (Math.random() - 0.5) * 4, vy: -Math.random() * 3 - 1, size: Math.random() * 2 + 1, life: 1 }); } xa.beginPath(); xa.moveTo(r.x, r.y); xa.lineTo(r.x, r.y + r.len); xa.globalAlpha = r.alpha; xa.stroke(); }); xa.globalAlpha = 1;
			xb.globalCompositeOperation = 'lighter'; splashes.forEach((s, i) => { s.x += s.vx; s.y += s.vy; s.vy += 0.15; s.life -= 0.02; if (s.life > 0) { xb.beginPath(); xb.arc(s.x, s.y, s.size * s.life, 0, Math.PI * 2); xb.fillStyle = `rgba(255,80,80,${s.life * 0.8})`; xb.fill(); } }); splashes.filter(s => s.life > 0); xb.globalCompositeOperation = 'source-over';
			const poolGrd = xc.createLinearGradient(0, h * 0.85, 0, h); poolGrd.addColorStop(0, 'transparent'); poolGrd.addColorStop(0.3, 'rgba(180,20,20,0.2)'); poolGrd.addColorStop(1, 'rgba(120,0,0,0.4)'); xc.fillStyle = poolGrd; xc.fillRect(0, h * 0.85, w, h * 0.15);
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s6', start: 'top 60%', onEnter: () => { on = true; const tl = gsap.timeline(); tl.to('#s6f', { opacity: 0.6, duration: 0.1 }, 0.5); tl.to('#s6f', { opacity: 0, duration: 2 }, 0.6); tl.to('#s6t', { opacity: 1, duration: 0.6 }, 1); tl.from('#s6t', { scale: 1.8, filter: 'blur(15px)', duration: 0.6, ease: 'power4.out' }, 1); tl.add(() => shake(document.getElementById('s6-content')!, 25, 0.5), 1); tl.to('#s6u', { opacity: 1, duration: 1 }, 1.8); tl.to('#s6l', { opacity: 0.12, duration: 0.5 }, 1.5); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initForge(); initConvergence(); initBloodbath(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="s4">
	<canvas bind:this={s4a}></canvas>
	<canvas bind:this={s4b} class="c2"></canvas>
	<div class="flash" id="s4f"></div>
	<div class="ct" id="s4-content">
		<h1 class="t1" id="s4t">FORGED</h1>
		<p class="t2" id="s4u">THE EDGE IS BUILT. NOT FOUND.</p>
	</div>
	<span class="lbl" id="s4l">04 — THE FORGE</span>
</section>
<div class="sp"></div>

<section class="hero" id="s5">
	<canvas bind:this={s5a}></canvas>
	<canvas bind:this={s5b} class="c2"></canvas>
	<div class="flash" id="s5f"></div>
	<div class="ct" id="s5-content">
		<h1 class="t1" id="s5t">CONVERGENCE</h1>
		<p class="t2" id="s5u">WHEN EVERYTHING ALIGNS</p>
	</div>
	<span class="lbl" id="s5l">05 — THE SETUP</span>
</section>
<div class="sp"></div>

<section class="hero" id="s6">
	<canvas bind:this={s6a}></canvas>
	<canvas bind:this={s6b} class="c2"></canvas>
	<canvas bind:this={s6c} class="c3"></canvas>
	<div class="flash" id="s6f"></div>
	<div class="ct" id="s6-content">
		<h1 class="t1" id="s6t">BLOODBATH</h1>
		<p class="t2" id="s6u">WHEN THEY SELL, WE HUNT</p>
	</div>
	<span class="lbl" id="s6l">06 — LIQUIDATION</span>
</section>

<style>
	:global(*) { margin: 0; padding: 0; box-sizing: border-box; }
	:global(html) { overflow-x: hidden; background: #000; }
	.hero { position: relative; width: 100%; height: 100vh; overflow: hidden; display: flex; align-items: center; justify-content: center; color: #fff; font-family: 'JetBrains Mono', monospace; }
	.hero canvas { position: absolute; inset: 0; width: 100%; height: 100%; z-index: 1; }
	.c2 { z-index: 2; } .c3 { z-index: 3; }
	.flash { position: absolute; inset: 0; z-index: 30; pointer-events: none; opacity: 0; }
	.ct { position: relative; z-index: 20; text-align: center; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%; pointer-events: none; }
	.lbl { position: absolute; top: 25px; left: 30px; z-index: 60; font-size: 0.45rem; letter-spacing: 0.6em; text-transform: uppercase; opacity: 0.15; }
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(4rem, 15vw, 14rem); line-height: 0.82; letter-spacing: -0.01em; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.4rem, 1vw, 0.65rem); letter-spacing: 0.8em; margin-top: 15px; opacity: 0; }
	.sp { height: 2px; background: #000; }
	#s4 { background: #0a0005; } #s4f { background: radial-gradient(ellipse at 50% 70%, rgba(255,150,0,0.7), transparent 40%); }
	#s4t { background: linear-gradient(180deg, #fff, #ff8800, #ff4400); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#s4u { color: rgba(255,150,50,0.5); } #s4l { color: #ff8800; }
	#s5 { background: #000a0a; } #s5f { background: #fff; }
	#s5t { color: #fff; text-shadow: 0 0 50px rgba(0,255,200,0.5); }
	#s5u { color: rgba(0,255,200,0.5); } #s5l { color: #00ffc8; }
	#s6 { background: #0a0000; } #s6f { background: rgba(255,0,0,0.4); }
	#s6t { color: #ff1133; text-shadow: 0 0 80px rgba(255,0,50,0.6); }
	#s6u { color: rgba(255,50,80,0.5); } #s6l { color: #ff1133; }
</style>
