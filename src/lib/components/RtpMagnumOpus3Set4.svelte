<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let s10a: HTMLCanvasElement, s10b: HTMLCanvasElement, s10c: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }
	function shake(el: HTMLElement, intensity = 20, dur = 0.5) { const s = Math.floor(dur / 0.016); for (let i = 0; i < s; i++) { const f = Math.pow(1 - i / s, 2); gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.6, delay: i * 0.016 }); } gsap.set(el, { x: 0, y: 0, delay: s * 0.016 }); }

	function initStrong() {
		const cv = [s10a, s10b, s10c];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s10a.width = s10a.parentElement!.offsetWidth, h = s10a.height = s10a.parentElement!.offsetHeight;
		let t = 0, on = false, formed = false;
		const pts: { x: number; y: number; tx: number; ty: number; vx: number; vy: number; size: number; hue: number }[] = [];
		let targets: { x: number; y: number }[] = [];
		function getTextPts(text: string, size: number) {
			const tc = document.createElement('canvas'); tc.width = w; tc.height = h;
			const tx = tc.getContext('2d')!;
			tx.font = `900 ${size}px Anton`; tx.textAlign = 'center'; tx.textBaseline = 'middle';
			tx.fillStyle = '#fff'; tx.fillText(text, w / 2, h / 2);
			const d = tx.getImageData(0, 0, w, h).data;
			const p: { x: number; y: number }[] = []; const step = 5;
			for (let y = 0; y < h; y += step) for (let x = 0; x < w; x += step) { if (d[(y * w + x) * 4 + 3] > 128) p.push({ x, y }); }
			return p;
		}
		function init() {
			targets = getTextPts('18,000+', Math.min(w * 0.2, 180));
			pts.length = 0;
			const count = Math.min(targets.length, 2000);
			for (let i = 0; i < count; i++) { pts.push({ x: Math.random() * w, y: Math.random() * h, tx: targets[i].x, ty: targets[i].y, vx: 0, vy: 0, size: Math.random() * 2.5 + 0.5, hue: 130 + Math.random() * 30 }); }
		}
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s10a.width; h = s10a.height; init(); }
		window.addEventListener('resize', resize);
		function draw() { if (!on) { rafIds.push(requestAnimationFrame(draw)); return; } t += 0.008; const [xa, xb, xc] = ctx; xa.fillStyle = 'rgba(0,0,0,0.1)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xc.clearRect(0, 0, w, h); pts.forEach(p => { const mdx = p.x - mx, mdy = p.y - my, md = Math.sqrt(mdx * mdx + mdy * mdy); if (md < 100) { p.vx += (mdx / md) * 6; p.vy += (mdy / md) * 6; } if (formed) { p.vx += (p.tx - p.x) * 0.06; p.vy += (p.ty - p.y) * 0.06; } p.vx *= 0.88; p.vy *= 0.88; p.x += p.vx; p.y += p.vy; const spd = Math.sqrt(p.vx * p.vx + p.vy * p.vy); xa.beginPath(); xa.arc(p.x, p.y, p.size, 0, Math.PI * 2); xa.fillStyle = `hsla(${p.hue},100%,60%,${0.3 + Math.min(spd * 0.05, 0.6)})`; if (spd > 2) { xa.shadowBlur = 4; xa.shadowColor = `hsla(${p.hue},100%,50%,0.3)`; } xa.fill(); xa.shadowBlur = 0; }); if (formed) { for (let i = 0; i < pts.length; i += 3) { for (let j = i + 1; j < pts.length; j += 3) { const dx = pts[i].x - pts[j].x, dy = pts[i].y - pts[j].y; if (dx * dx + dy * dy < 600) { xb.beginPath(); xb.moveTo(pts[i].x, pts[i].y); xb.lineTo(pts[j].x, pts[j].y); xb.strokeStyle = 'rgba(0,255,136,0.04)'; xb.lineWidth = 0.5; xb.stroke(); } } } } rafIds.push(requestAnimationFrame(draw)); }
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s10', start: 'top 60%', onEnter: () => { on = true; init(); timeouts.push(window.setTimeout(() => { formed = true; }, 500)); const tl = gsap.timeline({ delay: 3.5 }); tl.to('#s10f', { opacity: 0.2, duration: 0.05 }, 0); tl.to('#s10f', { opacity: 0, duration: 1 }, 0.05); tl.to('#s10t', { opacity: 1, duration: 0.5 }, 0.3); tl.from('#s10t', { scale: 3, filter: 'blur(20px)', duration: 0.5, ease: 'power4.out' }, 0.3); tl.add(() => shake(document.getElementById('s10-content')!, 20, 0.4), 0.3); tl.to('#s10u', { opacity: 1, duration: 1 }, 1); tl.to('#s10v', { opacity: 1, duration: 1 }, 1.8); tl.to('#s10l', { opacity: 0.15, duration: 0.5 }, 0.5); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initStrong(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="s10">
	<canvas bind:this={s10a}></canvas>
	<canvas bind:this={s10b} class="c2"></canvas>
	<canvas bind:this={s10c} class="c3"></canvas>
	<div class="flash" id="s10f"></div>
	<div class="ct" id="s10-content">
		<h1 class="t1" id="s10t">18,000 STRONG</h1>
		<p class="t2" id="s10u">THE REVOLUTION IS HERE</p>
		<p class="t3" id="s10v">JOIN THE COMMUNITY</p>
	</div>
	<span class="lbl" id="s10l">10 — THE MOVEMENT</span>
</section>

<style>
	:global(*) { margin: 0; padding: 0; box-sizing: border-box; }
	:global(html) { overflow-x: hidden; background: #000; }
	.hero { position: relative; width: 100%; height: 100vh; overflow: hidden; display: flex; align-items: center; justify-content: center; color: #fff; font-family: 'JetBrains Mono', monospace; }
	.hero canvas { position: absolute; inset: 0; width: 100%; height: 100%; z-index: 1; }
	.c2 { z-index: 2; } .c3 { z-index: 3; }
	.flash { position: absolute; inset: 0; z-index: 30; pointer-events: none; opacity: 0; }
	.ct { position: relative; z-index: 20; text-align: center; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%; pointer-events: none; }
	.lbl { position: absolute; top: 25px; left: 30px; z-index: 60; font-size: 0.38rem; letter-spacing: 0.7em; text-transform: uppercase; opacity: 0.15; }
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(3.5rem, 13vw, 12rem); line-height: 0.85; letter-spacing: -0.01em; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.4rem, 1vw, 0.65rem); letter-spacing: 0.7em; margin-top: 15px; opacity: 0; }
	.t3 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.4rem, 1vw, 0.65rem); letter-spacing: 0.5em; margin-top: 10px; opacity: 0; }
	#s10 { background: #000; }
	#s10t { color: #00ff88; text-shadow: 0 0 80px rgba(0,255,136,0.5); }
	#s10u { color: rgba(0,255,136,0.6); }
	#s10v { color: rgba(0,255,136,0.4); }
	#s10l { color: #00ff88; }
</style>
