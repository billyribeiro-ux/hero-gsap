<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let s7a: HTMLCanvasElement, s7b: HTMLCanvasElement, s7c: HTMLCanvasElement;
	let s8a: HTMLCanvasElement, s8b: HTMLCanvasElement, s8c: HTMLCanvasElement;
	let s9a: HTMLCanvasElement, s9b: HTMLCanvasElement, s9c: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }
	function shake(el: HTMLElement, intensity = 20, dur = 0.5) { const s = Math.floor(dur / 0.016); for (let i = 0; i < s; i++) { const f = Math.pow(1 - i / s, 2); gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.6, delay: i * 0.016 }); } gsap.set(el, { x: 0, y: 0, delay: s * 0.016 }); }

	function initNerveCenter() {
		const cv = [s7a, s7b, s7c];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s7a.width = s7a.parentElement!.offsetWidth, h = s7a.height = s7a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const charts: { x: number; y: number; w: number; h: number; data: number[]; color: string }[] = [];
		for (let i = 0; i < 6; i++) { const cw = w * 0.25, ch = h * 0.2, cx = (i % 3) * cw * 1.1 + w * 0.1, cy = Math.floor(i / 3) * ch * 1.3 + h * 0.15; const data: number[] = []; for (let j = 0; j < 20; j++) data.push(0.3 + Math.random() * 0.4); charts.push({ x: cx, y: cy, w: cw, h: ch, data, color: i % 2 === 0 ? '0,255,136' : '255,220,50' }); }
		const particles: { x: number; y: number; vx: number; vy: number; life: number; char: string }[] = [];
		const chars = '0123456789$€¥';
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s7a.width; h = s7a.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.016; const [xa, xb, xc] = ctx;
			xa.fillStyle = 'rgba(0,5,5,0.1)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xc.clearRect(0, 0, w, h);
			xa.strokeStyle = 'rgba(0,255,136,0.15)'; xa.lineWidth = 1; for (let i = 0; i < 5; i++) { const y = h * (0.2 + i * 0.15); xa.beginPath(); xa.moveTo(0, y); xa.lineTo(w, y); xa.stroke(); } for (let i = 0; i < 8; i++) { const x = w * (0.1 + i * 0.11); xa.beginPath(); xa.moveTo(x, 0); xa.lineTo(x, h); xa.stroke(); }
			charts.forEach((c, idx) => { xb.strokeStyle = `rgba(${c.color},0.8)`; xb.lineWidth = 2; xb.beginPath(); c.data.forEach((d, i) => { const x = c.x + i / c.data.length * c.w, y = c.y + c.h - d * c.h; if (i === 0) xb.moveTo(x, y); else xb.lineTo(x, y); }); xb.stroke(); xb.fillStyle = `rgba(${c.color},0.1)`; xb.fillRect(c.x, c.y, c.w, c.h); xb.strokeStyle = `rgba(${c.color},0.3)`; xb.strokeRect(c.x, c.y, c.w, c.h); });
			if (Math.random() > 0.8) particles.push({ x: Math.random() * w, y: Math.random() * h, vx: (Math.random() - 0.5) * 2, vy: -Math.random() * 3 - 1, life: 1, char: chars[Math.floor(Math.random() * chars.length)] });
			xc.fillStyle = 'rgba(0,255,136,0.8)'; xc.font = '10px "JetBrains Mono"'; particles.forEach((p, i) => { p.x += p.vx; p.y += p.vy; p.life -= 0.015; if (p.life > 0) { xc.globalAlpha = p.life; xc.fillText(p.char, p.x, p.y); } }); xc.globalAlpha = 1; particles.filter(p => p.life > 0);
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s7', start: 'top 60%', onEnter: () => { on = true; const tl = gsap.timeline(); tl.to('#s7t', { opacity: 1, duration: 0.6 }, 0); tl.from('#s7t', { scale: 1.5, filter: 'blur(10px)', duration: 0.6, ease: 'power4.out' }, 0); tl.to('#s7u', { opacity: 1, duration: 1 }, 0.8); tl.to('#s7l', { opacity: 0.12, duration: 0.5 }, 0.5); } }); triggers.push(trigger);
	}

	function initAscension() {
		const cv = [s8a, s8b, s8c];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s8a.width = s8a.parentElement!.offsetWidth, h = s8a.height = s8a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const clouds: { x: number; y: number; w: number; h: number; speed: number; alpha: number; layer: number }[] = [];
		for (let i = 0; i < 15; i++) clouds.push({ x: Math.random() * w, y: Math.random() * h, w: 100 + Math.random() * 200, h: 30 + Math.random() * 50, speed: 0.2 + Math.random() * 0.5, alpha: 0.1 + Math.random() * 0.3, layer: Math.floor(Math.random() * 3) });
		const rays: { angle: number; width: number; speed: number; alpha: number }[] = [];
		for (let i = 0; i < 8; i++) rays.push({ angle: Math.random() * Math.PI, width: 20 + Math.random() * 40, speed: 0.001 + Math.random() * 0.002, alpha: 0.05 + Math.random() * 0.1 });
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s8a.width; h = s8a.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; const [xa, xb, xc] = ctx;
			xa.fillStyle = 'rgba(0,3,10,0.08)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xc.clearRect(0, 0, w, h);
			const sunY = h * 0.3 + Math.sin(t * 0.5) * 20; const sunGrd = xa.createRadialGradient(w * 0.5, sunY, 0, w * 0.5, sunY, 200); sunGrd.addColorStop(0, 'rgba(255,220,150,0.3)'); sunGrd.addColorStop(0.5, 'rgba(255,200,100,0.1)'); sunGrd.addColorStop(1, 'transparent'); xa.fillStyle = sunGrd; xa.fillRect(0, 0, w, h);
			rays.forEach(r => { r.angle += r.speed; xb.globalCompositeOperation = 'lighter'; const grd = xb.createLinearGradient(w * 0.5 + Math.cos(r.angle) * 300, sunY + Math.sin(r.angle) * 300, w * 0.5 - Math.cos(r.angle) * 300, sunY - Math.sin(r.angle) * 300); grd.addColorStop(0, 'transparent'); grd.addColorStop(0.5, `rgba(255,240,200,${r.alpha})`); grd.addColorStop(1, 'transparent'); xb.fillStyle = grd; xb.fillRect(0, 0, w, h); }); xb.globalCompositeOperation = 'source-over';
			clouds.forEach(c => { c.y -= c.speed; if (c.y < -100) c.y = h + 100; const ctx = c.layer === 0 ? xa : c.layer === 1 ? xb : xc; ctx.fillStyle = `rgba(255,255,255,${c.alpha * 0.3})`; ctx.beginPath(); ctx.ellipse(c.x, c.y, c.w, c.h, 0, 0, Math.PI * 2); ctx.fill(); });
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s8', start: 'top 60%', onEnter: () => { on = true; const tl = gsap.timeline(); tl.to('#s8t', { opacity: 1, duration: 0.6 }, 0); tl.from('#s8t', { y: 100, filter: 'blur(20px)', duration: 0.6, ease: 'power4.out' }, 0); tl.to('#s8u', { opacity: 1, duration: 1 }, 0.8); tl.to('#s8l', { opacity: 0.12, duration: 0.5 }, 0.5); } }); triggers.push(trigger);
	}

	function initOmega() {
		const cv = [s9a, s9b, s9c];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s9a.width = s9a.parentElement!.offsetWidth, h = s9a.height = s9a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const bursts: { x: number; y: number; r: number; maxR: number; alpha: number; hue: number }[] = [];
		const chains: { x: number; y: number; vy: number; color: string; strike: number }[] = [];
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s9a.width; h = s9a.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.016; const [xa, xb, xc] = ctx;
			xa.fillStyle = 'rgba(5,0,5,0.12)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xc.clearRect(0, 0, w, h);
			if (Math.random() > 0.9) { const hue = Math.random() > 0.5 ? 320 : 30; bursts.push({ x: Math.random() * w, y: Math.random() * h, r: 0, maxR: 50 + Math.random() * 100, alpha: 0.8, hue }); gsap.to(bursts[bursts.length - 1], { r: bursts[bursts.length - 1].maxR, alpha: 0, duration: 1.5, ease: 'power2.out' }); }
			xa.globalCompositeOperation = 'lighter'; bursts.forEach((b, i) => { if (b.alpha > 0.01) { const grd = xa.createRadialGradient(b.x, b.y, 0, b.x, b.y, b.r); grd.addColorStop(0, `hsla(${b.hue},100%,70%,${b.alpha})`); grd.addColorStop(0.5, `hsla(${b.hue},80%,50%,${b.alpha * 0.5})`); grd.addColorStop(1, 'transparent'); xa.fillStyle = grd; xa.beginPath(); xa.arc(b.x, b.y, b.r, 0, Math.PI * 2); xa.fill(); } }); bursts.filter(b => b.alpha > 0.01); xa.globalCompositeOperation = 'source-over';
			if (Math.random() > 0.95) chains.push({ x: Math.random() * w, y: h + 20, vy: -Math.random() * 8 - 4, color: Math.random() > 0.5 ? '#ff00cc' : '#ff6600', strike: Math.random() });
			xb.strokeStyle = 'rgba(255,255,255,0.8)'; xb.lineWidth = 2; chains.forEach((c, i) => { c.y += c.vy; if (c.y > -50) { xb.beginPath(); xb.moveTo(c.x, c.y); xb.lineTo(c.x, c.y - 30); xb.stroke(); xb.fillStyle = c.color; xb.fillRect(c.x - 8, c.y - 35, 16, 10); } }); chains.filter(c => c.y > -50);
			xc.fillStyle = 'rgba(255,200,100,0.8)'; xc.font = 'bold 12px "JetBrains Mono"'; xc.textAlign = 'center'; xc.fillText('GAMMA', w * 0.3, h * 0.15); xc.fillStyle = 'rgba(255,100,200,0.8)'; xc.fillText('DELTA', w * 0.5, h * 0.15); xc.fillStyle = 'rgba(100,255,150,0.8)'; xc.fillText('THETA', w * 0.7, h * 0.15);
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s9', start: 'top 60%', onEnter: () => { on = true; const tl = gsap.timeline(); tl.to('#s9f', { opacity: 0.6, duration: 0.1 }, 0.5); tl.to('#s9f', { opacity: 0, duration: 2 }, 0.6); tl.to('#s9t', { opacity: 1, duration: 0.6 }, 1); tl.from('#s9t', { scale: 1.5, filter: 'blur(15px)', duration: 0.6, ease: 'power4.out' }, 1); tl.add(() => shake(document.getElementById('s9-content')!, 30, 0.6), 1); tl.to('#s9u', { opacity: 1, duration: 1 }, 1.8); tl.to('#s9v', { opacity: 1, duration: 1 }, 2.2); tl.to('#s9l', { opacity: 0.12, duration: 0.5 }, 1.5); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initNerveCenter(); initAscension(); initOmega(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="s7">
	<canvas bind:this={s7a}></canvas>
	<canvas bind:this={s7b} class="c2"></canvas>
	<canvas bind:this={s7c} class="c3"></canvas>
	<div class="flash" id="s7f"></div>
	<div class="ct" id="s7-content">
		<h1 class="t1" id="s7t">NERVE CENTER</h1>
		<p class="t2" id="s7u">TOTAL MARKET AWARENESS</p>
	</div>
	<span class="lbl" id="s7l">07 — COMMAND</span>
</section>
<div class="sp"></div>

<section class="hero" id="s8">
	<canvas bind:this={s8a}></canvas>
	<canvas bind:this={s8b} class="c2"></canvas>
	<canvas bind:this={s8c} class="c3"></canvas>
	<div class="flash" id="s8f"></div>
	<div class="ct" id="s8-content">
		<h1 class="t1" id="s8t">ASCENSION</h1>
		<p class="t2" id="s8u">ABOVE ALL RESISTANCE</p>
	</div>
	<span class="lbl" id="s8l">08 — BREAKOUT</span>
</section>
<div class="sp"></div>

<section class="hero" id="s9">
	<canvas bind:this={s9a}></canvas>
	<canvas bind:this={s9b} class="c2"></canvas>
	<canvas bind:this={s9c} class="c3"></canvas>
	<div class="flash" id="s9f"></div>
	<div class="ct" id="s9-content">
		<h1 class="t1" id="s9t">OMEGA</h1>
		<p class="t2" id="s9u">GAMMA × DELTA × THETA = DESTRUCTION</p>
		<p class="t3" id="s9v">[ 0DTE SPX — 573× RETURN ]</p>
	</div>
	<span class="lbl" id="s9l">09 — OPTIONS NUKE</span>
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
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(3rem, 10vw, 9rem); line-height: 0.82; letter-spacing: -0.01em; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.4rem, 1vw, 0.65rem); letter-spacing: 0.8em; margin-top: 15px; opacity: 0; }
	.t3 { font-family: 'JetBrains Mono', monospace; font-size: 0.45rem; letter-spacing: 0.4em; margin-top: 25px; opacity: 0; }
	.sp { height: 2px; background: #000; }
	#s7 { background: #000505; } #s7f { background: rgba(0,255,136,0.3); }
	#s7t { color: #00ff88; text-shadow: 0 0 60px rgba(0,255,136,0.5); }
	#s7u { color: rgba(0,255,136,0.4); } #s7l { color: #00ff88; }
	#s8 { background: #000308; } #s8f { background: linear-gradient(0deg, transparent, rgba(255,255,200,0.6)); }
	#s8t { background: linear-gradient(180deg, #fff, #ffd700, #00ff88); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#s8u { color: rgba(255,220,100,0.5); } #s8l { color: #ffd700; }
	#s9 { background: #050005; } #s9f { background: radial-gradient(circle, rgba(255,0,200,0.6), transparent 40%); }
	#s9t { background: linear-gradient(90deg, #ff00cc, #ff6600, #ffd700); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#s9u { color: rgba(255,0,200,0.5); }
	#s9v { color: rgba(255,100,0,0.3); }
	#s9l { color: #ff00cc; }
</style>
