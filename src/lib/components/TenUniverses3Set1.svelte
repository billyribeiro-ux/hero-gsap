<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let c1: HTMLCanvasElement, c2: HTMLCanvasElement, c3: HTMLCanvasElement;
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

	function initSupernova() {
		const x = c1.getContext('2d')!;
		let w = c1.width = c1.parentElement!.offsetWidth, h = c1.height = c1.parentElement!.offsetHeight;
		const stars: { x: number; y: number; z: number; size: number }[] = [], gas: { x: number; y: number; r: number; vx: number; vy: number; hue: number; alpha: number }[] = [];
		let t = 0, active = false, exploded = false;
		for (let i = 0; i < 800; i++) stars.push({ x: Math.random() * 2 - 1, y: Math.random() * 2 - 1, z: Math.random(), size: Math.random() * 1.5 + 0.3 });
		for (let i = 0; i < 60; i++) gas.push({ x: (Math.random() - 0.5) * w, y: (Math.random() - 0.5) * h, r: Math.random() * 150 + 50, vx: (Math.random() - 0.5) * 0.3, vy: (Math.random() - 0.5) * 0.3, hue: Math.random() * 60 - 10, alpha: 0 });
		function resize() { w = c1.width = c1.parentElement!.offsetWidth; h = c1.height = c1.parentElement!.offsetHeight; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.005; x.fillStyle = 'rgba(2,0,5,0.15)'; x.fillRect(0, 0, w, h);
			const cx = w / 2, cy = h / 2, mouseX = (mx - cx) / w, mouseY = (my - cy) / h;
			stars.forEach(s => {
				const px = cx + (s.x + mouseX * s.z * 0.3) * w * 0.6, py = cy + (s.y + mouseY * s.z * 0.3) * h * 0.6;
				const flicker = 0.5 + Math.sin(t * 10 + s.x * 100) * 0.5, bright = s.z * flicker;
				x.beginPath(); x.arc(px, py, s.size * s.z, 0, Math.PI * 2);
				x.fillStyle = `rgba(255,${200 + Math.floor(s.z * 55)},${150 + Math.floor(s.z * 105)},${bright * 0.8})`; x.fill();
				if (s.z > 0.8) { x.shadowBlur = 8; x.shadowColor = `rgba(255,200,150,${bright * 0.3})`; x.fill(); x.shadowBlur = 0; }
			});
			const coreR = 30 + Math.sin(t * 3) * 10, grd = x.createRadialGradient(cx, cy, 0, cx, cy, coreR * 4);
			grd.addColorStop(0, `rgba(255,220,180,${0.3 + Math.sin(t * 5) * 0.1})`); grd.addColorStop(0.3, 'rgba(255,120,20,0.1)'); grd.addColorStop(1, 'transparent');
			x.fillStyle = grd; x.fillRect(cx - coreR * 4, cy - coreR * 4, coreR * 8, coreR * 8);
			x.beginPath(); x.arc(cx, cy, coreR * 0.3, 0, Math.PI * 2);
			x.fillStyle = `rgba(255,255,240,${0.6 + Math.sin(t * 8) * 0.2})`; x.shadowBlur = 40; x.shadowColor = 'rgba(255,180,80,0.8)'; x.fill(); x.shadowBlur = 0;
			x.beginPath(); x.arc(cx, cy, 120 + Math.sin(t * 2) * 15, 0, Math.PI * 2);
			x.strokeStyle = `rgba(255,150,50,${0.05 + Math.sin(t * 3) * 0.03})`; x.lineWidth = 2; x.stroke();
			gas.forEach(g => { g.x += g.vx; g.y += g.vy; if (exploded && g.alpha < 0.15) g.alpha += 0.001; x.beginPath(); x.arc(cx + g.x + mouseX * 30, cy + g.y + mouseY * 30, g.r, 0, Math.PI * 2); x.fillStyle = `hsla(${g.hue + t * 20},80%,50%,${g.alpha})`; x.fill(); });
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s1', start: 'top 80%', onEnter: () => { active = true; const tl = gsap.timeline({ delay: 0.3 }); tl.add(() => { exploded = true; }, 1); tl.fromTo('.s1-vignette', { background: 'radial-gradient(ellipse,rgba(255,200,100,0.8) 0%,transparent 60%)' }, { background: 'radial-gradient(ellipse,transparent 30%,rgba(0,0,5,0.8) 100%)', duration: 1.5, ease: 'power3.out' }, 1); tl.to('#s1t', { opacity: 1, duration: 0.8, ease: 'power4.out' }, 1.5); tl.add(() => shake(document.getElementById('s1-content')!, 25, 0.5), 1.5); tl.to('#s1s', { opacity: 1, duration: 1 }, 2.2); } });
		triggers.push(trigger);
	}

	function initNeural() {
		const x = c2.getContext('2d')!; let w = c2.width = c2.parentElement!.offsetWidth, h = c2.height = c2.parentElement!.offsetHeight;
		const nodes: { x: number; y: number; connections: number[]; size: number; pulse: number; baseAlpha: number }[] = [], pulses: { from: number; to: number; progress: number; speed: number }[] = [];
		let t = 0, active = false;
		for (let i = 0; i < 120; i++) nodes.push({ x: Math.random() * w, y: Math.random() * h, connections: [], size: Math.random() * 3 + 1.5, pulse: 0, baseAlpha: Math.random() * 0.3 + 0.1 });
		nodes.forEach((n, i) => nodes.forEach((m, j) => { if (i >= j) return; const d = Math.hypot(n.x - m.x, n.y - m.y); if (d < 200) n.connections.push(j); }));
		function firePulse() { if (!active) return; const src = Math.floor(Math.random() * nodes.length), n = nodes[src]; if (n.connections.length === 0) return; const tgt = n.connections[Math.floor(Math.random() * n.connections.length)]; pulses.push({ from: src, to: tgt, progress: 0, speed: 0.02 + Math.random() * 0.03 }); nodes[src].pulse = 1; timeouts.push(window.setTimeout(firePulse, Math.random() * 300 + 50)); }
		function resize() { w = c2.width = c2.parentElement!.offsetWidth; h = c2.height = c2.parentElement!.offsetHeight; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.01; x.fillStyle = 'rgba(0,5,8,0.08)'; x.fillRect(0, 0, w, h);
			const mouseX = (mx / w - 0.5) * 40, mouseY = (my / h - 0.5) * 40;
			nodes.forEach((n, i) => n.connections.forEach(j => { const m = nodes[j]; x.beginPath(); x.moveTo(n.x + mouseX * 0.02 * (i % 3), n.y + mouseY * 0.02 * (i % 3)); x.lineTo(m.x + mouseX * 0.02 * (j % 3), m.y + mouseY * 0.02 * (j % 3)); x.strokeStyle = 'rgba(0,180,255,0.04)'; x.lineWidth = 0.5; x.stroke(); }));
			pulses.forEach(p => { p.progress += p.speed; const n = nodes[p.from], m = nodes[p.to], px = n.x + (m.x - n.x) * p.progress + mouseX * 0.02, py = n.y + (m.y - n.y) * p.progress + mouseY * 0.02; x.beginPath(); x.arc(px, py, 3, 0, Math.PI * 2); x.fillStyle = `rgba(0,220,255,${1 - p.progress})`; x.shadowBlur = 15; x.shadowColor = '#00ddff'; x.fill(); x.shadowBlur = 0; if (p.progress >= 1) nodes[p.to].pulse = 1; });
			pulses.filter(p => p.progress < 1); nodes.forEach((n, i) => { n.pulse *= 0.95; const md = Math.hypot(mx - n.x, my - n.y), mouseGlow = Math.max(0, 1 - md / 200) * 0.4, alpha = n.baseAlpha + n.pulse * 0.7 + mouseGlow; x.beginPath(); x.arc(n.x + mouseX * 0.01 * (i % 5), n.y + mouseY * 0.01 * (i % 5), n.size * (1 + n.pulse * 2), 0, Math.PI * 2); x.fillStyle = `rgba(0,200,255,${alpha})`; if (n.pulse > 0.3 || mouseGlow > 0.1) { x.shadowBlur = 12; x.shadowColor = `rgba(0,220,255,${alpha})`; } x.fill(); x.shadowBlur = 0; });
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s2', start: 'top 80%', onEnter: () => { active = true; firePulse(); const tl = gsap.timeline({ delay: 0.5 }); tl.to('#s2t', { opacity: 1, duration: 0.6, ease: 'power4.out' }, 0.5); tl.to('#s2s', { opacity: 1, duration: 1 }, 1.2); } }); triggers.push(trigger);
	}

	function initAbyss() {
		const x = c3.getContext('2d')!; let w = c3.width = c3.parentElement!.offsetWidth, h = c3.height = c3.parentElement!.offsetHeight;
		const orgs: { x: number; y: number; vx: number; vy: number; size: number; hue: number; pulse: number; pulseSpeed: number; trail: { x: number; y: number }[] }[] = [], rings: { x: number; y: number; r: number; alpha: number }[] = [];
		let t = 0, active = false;
		for (let i = 0; i < 200; i++) orgs.push({ x: Math.random() * w, y: Math.random() * h, vx: (Math.random() - 0.5) * 0.3, vy: -Math.random() * 0.5 - 0.1, size: Math.random() * 4 + 1, hue: 140 + Math.random() * 40, pulse: Math.random() * Math.PI * 2, pulseSpeed: 0.02 + Math.random() * 0.03, trail: [] });
		function sonarPing() { if (!active) return; rings.push({ x: w / 2, y: h / 2, r: 0, alpha: 0.3 }); timeouts.push(window.setTimeout(sonarPing, 3000 + Math.random() * 2000)); }
		function resize() { w = c3.width = c3.parentElement!.offsetWidth; h = c3.height = c3.parentElement!.offsetHeight; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; x.fillStyle = 'rgba(0,2,5,0.06)'; x.fillRect(0, 0, w, h);
			const cx = mx / w - 0.5, cy = my / h - 0.5; rings.forEach(r => { r.r += 2; r.alpha *= 0.98; x.beginPath(); x.arc(r.x, r.y, r.r, 0, Math.PI * 2); x.strokeStyle = `rgba(0,255,170,${r.alpha})`; x.lineWidth = 1; x.stroke(); }); rings.filter(r => r.alpha > 0.01);
			orgs.forEach(o => { o.pulse += o.pulseSpeed; o.vx += cx * 0.01; o.vy += cy * 0.005; o.vx *= 0.99; o.vy *= 0.99; o.x += o.vx; o.y += o.vy; if (o.x < -20) o.x = w + 20; if (o.x > w + 20) o.x = -20; if (o.y < -20) o.y = h + 20; if (o.y > h + 20) o.y = -20; o.trail.push({ x: o.x, y: o.y }); if (o.trail.length > 8) o.trail.shift(); const glow = 0.3 + Math.sin(o.pulse) * 0.3; o.trail.forEach((tp, i) => { const a = (i / o.trail.length) * glow * 0.3; x.beginPath(); x.arc(tp.x, tp.y, o.size * 0.5 * (i / o.trail.length), 0, Math.PI * 2); x.fillStyle = `hsla(${o.hue},100%,60%,${a})`; x.fill(); }); x.beginPath(); x.arc(o.x, o.y, o.size * (0.8 + Math.sin(o.pulse) * 0.2), 0, Math.PI * 2); x.fillStyle = `hsla(${o.hue},100%,70%,${glow})`; x.shadowBlur = o.size * 4; x.shadowColor = `hsla(${o.hue},100%,50%,${glow * 0.5})`; x.fill(); x.shadowBlur = 0; });
			for (let i = 0; i < 5; i++) { const rx = w * 0.15 + i * w * 0.18 + Math.sin(t + i) * 30; const grd = x.createLinearGradient(rx, 0, rx + 20, h * 0.6); grd.addColorStop(0, `rgba(0,100,120,${0.01 + Math.sin(t + i * 0.5) * 0.005})`); grd.addColorStop(1, 'transparent'); x.fillStyle = grd; x.fillRect(rx - 20, 0, 60, h * 0.6); }
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s3', start: 'top 80%', onEnter: () => { active = true; sonarPing(); const tl = gsap.timeline({ delay: 0.5 }); tl.to('#s3t', { opacity: 1, duration: 1.2, ease: 'power2.out' }, 0.5); tl.to('#s3s', { opacity: 1, duration: 1 }, 1.5); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initSupernova(); initNeural(); initAbyss(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero h-supernova" id="s1">
	<canvas bind:this={c1}></canvas>
	<div class="vignette s1-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s1-content">
		<h1 class="t1" id="s1t">SUPERNOVA</h1>
		<p class="t2" id="s1s">WHEN STARS COLLAPSE, LEGENDS ARE BORN</p>
	</div>
	<span class="label">01 — SUPERNOVA</span>
	<span class="num">01</span>
</section>
<div class="divider"></div>

<section class="hero h-neural" id="s2">
	<canvas bind:this={c2}></canvas>
	<div class="vignette s2-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s2-content">
		<h1 class="t1" id="s2t">SYNAPSE</h1>
		<p class="t2" id="s2s">PATTERN RECOGNITION AT NEURAL SPEED</p>
	</div>
	<span class="label">02 — NEURAL</span>
	<span class="num">02</span>
</section>
<div class="divider"></div>

<section class="hero h-abyss" id="s3">
	<canvas bind:this={c3}></canvas>
	<div class="vignette s3-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s3-content">
		<h1 class="t1" id="s3t">ABYSS</h1>
		<p class="t2" id="s3s">DEPTH IS WHERE THE EDGE LIVES</p>
	</div>
	<span class="label">03 — ABYSS</span>
	<span class="num">03</span>
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
	.h-supernova { background: #020005; }
	.h-neural { background: #000508; }
	.h-abyss { background: #000205; }
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(4rem, 14vw, 12rem); line-height: 0.85; letter-spacing: 0.02em; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.5rem, 1.2vw, 0.75rem); letter-spacing: 0.6em; margin-top: 20px; opacity: 0; }
	.divider { height: 5vh; background: #000; }
	.s1-vignette { background: radial-gradient(ellipse, transparent 30%, rgba(0,0,5,0.8) 100%); }
	.s2-vignette { background: radial-gradient(ellipse, transparent 40%, rgba(0,5,8,0.8) 100%); }
	.s3-vignette { background: radial-gradient(ellipse, transparent 30%, rgba(0,2,5,0.9) 100%); }
	#s1t { background: linear-gradient(180deg, #fff 0%, #ff8800 40%, #ff2200 100%); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#s1s { color: #ff8800; }
	#s2t { color: #00ddff; text-shadow: 0 0 60px rgba(0,200,255,0.4); }
	#s2s { color: rgba(0,200,255,0.5); }
	#s3t { color: #00ffaa; text-shadow: 0 0 80px rgba(0,255,170,0.3); }
	#s3s { color: rgba(0,255,170,0.4); }
	.h-supernova .label { color: #ff8800; }
	.h-supernova .num { color: #ff4400; }
	.h-neural .label { color: #00ccff; }
	.h-neural .num { color: #00ccff; }
	.h-abyss .label { color: #00ffaa; }
	.h-abyss .num { color: #00aa88; }
</style>
