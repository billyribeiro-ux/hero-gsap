<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let c7: HTMLCanvasElement, c8: HTMLCanvasElement, c9: HTMLCanvasElement;
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

	function initPredator() {
		const x = c7.getContext('2d')!; let w = c7.width = c7.parentElement!.offsetWidth, h = c7.height = c7.parentElement!.offsetHeight;
		let t = 0, active = false; const targets: { x: number; y: number; size: number; locked: boolean; heat: number }[] = [];
		for (let i = 0; i < 8; i++) targets.push({ x: Math.random() * w * 0.6 + w * 0.2, y: Math.random() * h * 0.6 + h * 0.2, size: Math.random() * 40 + 20, locked: false, heat: Math.random() * 0.5 + 0.5 });
		function resize() { w = c7.width = c7.parentElement!.offsetWidth; h = c7.height = c7.parentElement!.offsetHeight; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.01; x.fillStyle = 'rgba(2,2,0,0.1)'; x.fillRect(0, 0, w, h);
			const scanAngle = t * 0.8, scanX = w / 2 + Math.cos(scanAngle) * w * 0.4;
			const scanGrd = x.createLinearGradient(w / 2, h / 2, scanX, h / 2 + Math.sin(scanAngle) * h * 0.3);
			scanGrd.addColorStop(0, 'transparent'); scanGrd.addColorStop(1, 'rgba(255,200,0,0.02)');
			x.fillStyle = scanGrd; x.beginPath(); x.moveTo(w / 2, h / 2); x.arc(w / 2, h / 2, w * 0.5, scanAngle - 0.3, scanAngle + 0.3); x.closePath(); x.fill();
			for (let i = 0; i < 50; i++) { const nx = Math.random() * w, ny = Math.random() * h; x.fillStyle = `rgba(${Math.random() * 50 + 20},${Math.random() * 30 + 10},0,0.02)`; x.fillRect(nx, ny, Math.random() * 20 + 5, Math.random() * 20 + 5); }
			targets.forEach(tgt => { const grd = x.createRadialGradient(tgt.x, tgt.y, 0, tgt.x, tgt.y, tgt.size); grd.addColorStop(0, `rgba(255,200,0,${tgt.heat * 0.15 + Math.sin(t * 3) * 0.03})`); grd.addColorStop(0.5, `rgba(255,100,0,${tgt.heat * 0.08})`); grd.addColorStop(1, 'transparent'); x.fillStyle = grd; x.fillRect(tgt.x - tgt.size, tgt.y - tgt.size, tgt.size * 2, tgt.size * 2); });
			const bs = 40 + Math.sin(t * 5) * 5; x.strokeStyle = `rgba(255,200,0,${0.5 + Math.sin(t * 8) * 0.2})`; x.lineWidth = 2;
			x.beginPath(); x.moveTo(mx - bs, my - bs + 12); x.lineTo(mx - bs, my - bs); x.lineTo(mx - bs + 12, my - bs); x.stroke();
			x.beginPath(); x.moveTo(mx + bs - 12, my - bs); x.lineTo(mx + bs, my - bs); x.lineTo(mx + bs, my - bs + 12); x.stroke();
			x.beginPath(); x.moveTo(mx + bs, my + bs - 12); x.lineTo(mx + bs, my + bs); x.lineTo(mx + bs - 12, my + bs); x.stroke();
			x.beginPath(); x.moveTo(mx - bs + 12, my + bs); x.lineTo(mx - bs, my + bs); x.lineTo(mx - bs, my + bs - 12); x.stroke();
			x.beginPath(); x.moveTo(mx, my - 8); x.lineTo(mx + 8, my); x.lineTo(mx, my + 8); x.lineTo(mx - 8, my); x.closePath(); x.strokeStyle = `rgba(255,200,0,${0.3 + Math.sin(t * 6) * 0.2})`; x.lineWidth = 1; x.stroke();
			x.fillStyle = 'rgba(255,200,0,0.3)'; x.font = '9px "JetBrains Mono"'; x.fillText(`DIST: ${(Math.hypot(mx - w / 2, my - h / 2) / 10).toFixed(1)}m`, mx + bs + 8, my - bs + 10); x.fillText(`HEAT: ${(0.5 + Math.sin(t * 2) * 0.3).toFixed(2)}`, mx + bs + 8, my - bs + 25);
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s7', start: 'top 80%', onEnter: () => { active = true; const tl = gsap.timeline({ delay: 0.3 }); tl.to('#s7t', { opacity: 1, duration: 0.1 }, 1); tl.add(() => shake(document.getElementById('s7-content')!, 15, 0.3), 1); tl.to('#s7s', { opacity: 1, duration: 0.8 }, 1.5); } }); triggers.push(trigger);
	}

	function initTimewarp() {
		const x = c8.getContext('2d')!; let w = c8.width = c8.parentElement!.offsetWidth, h = c8.height = c8.parentElement!.offsetHeight;
		let t = 0, active = false; const lines: { angle: number; dist: number; speed: number; width: number; len: number; alpha: number }[] = [], fragments: { text: string; x: number; y: number; vx: number; vy: number; rotation: number; rotSpeed: number; alpha: number }[] = [];
		for (let i = 0; i < 200; i++) { const angle = Math.random() * Math.PI * 2; lines.push({ angle, dist: Math.random() * Math.max(w, h) * 0.7 + 100, speed: Math.random() * 8 + 3, width: Math.random() * 2 + 0.5, len: Math.random() * 60 + 20, alpha: Math.random() * 0.4 + 0.1 }); }
		for (let i = 1; i <= 12; i++) { const angle = (i / 12) * Math.PI * 2 - Math.PI / 2, r = Math.min(w, h) * 0.3; fragments.push({ text: String(i), x: Math.cos(angle) * r, y: Math.sin(angle) * r, vx: (Math.random() - 0.5) * 3, vy: (Math.random() - 0.5) * 3, rotation: 0, rotSpeed: (Math.random() - 0.5) * 0.05, alpha: 0 }); }
		function resize() { w = c8.width = c8.parentElement!.offsetWidth; h = c8.height = c8.parentElement!.offsetHeight; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.015; x.fillStyle = 'rgba(2,0,10,0.1)'; x.fillRect(0, 0, w, h);
			const cx = w / 2 + (mx / w - 0.5) * 40, cy = h / 2 + (my / h - 0.5) * 40;
			lines.forEach(l => { l.dist -= l.speed; if (l.dist < 10) { l.dist = Math.max(w, h) * 0.8; l.angle = Math.random() * Math.PI * 2; } const ex = cx + Math.cos(l.angle) * l.dist, ey = cy + Math.sin(l.angle) * l.dist, sx = cx + Math.cos(l.angle) * (l.dist + l.len), sy = cy + Math.sin(l.angle) * (l.dist + l.len); const fade = Math.min(1, l.dist / 200); x.beginPath(); x.moveTo(sx, sy); x.lineTo(ex, ey); x.strokeStyle = `rgba(200,100,255,${l.alpha * fade})`; x.lineWidth = l.width; x.stroke(); });
			for (let i = 0; i < 3; i++) { const r = 20 + i * 25 + Math.sin(t * 3 + i) * 10; x.beginPath(); x.arc(cx, cy, r, t * 2 + i, t * 2 + i + Math.PI * 1.5); x.strokeStyle = `rgba(200,100,255,${0.1 - i * 0.025})`; x.lineWidth = 2; x.stroke(); }
			fragments.forEach(f => { f.rotation += f.rotSpeed; const fx = cx + f.x * Math.cos(t * 0.3) - f.y * Math.sin(t * 0.3), fy = cy + f.x * Math.sin(t * 0.3) + f.y * Math.cos(t * 0.3); x.save(); x.translate(fx, fy); x.rotate(f.rotation); x.fillStyle = `rgba(200,100,255,${f.alpha})`; x.font = 'bold 20px Orbitron'; x.textAlign = 'center'; x.fillText(f.text, 0, 0); x.restore(); });
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s8', start: 'top 80%', onEnter: () => { active = true; fragments.forEach((f, i) => { gsap.to(f, { alpha: 0.3, duration: 0.5, delay: i * 0.08 }); }); const tl = gsap.timeline({ delay: 0.5 }); tl.to('#s8t', { opacity: 1, duration: 0.5, ease: 'power4.out' }, 0.8); tl.to('#s8s', { opacity: 1, duration: 1 }, 1.5); } }); triggers.push(trigger);
	}

	function initVolcanic() {
		const x = c9.getContext('2d')!; let w = c9.width = c9.parentElement!.offsetWidth, h = c9.height = c9.parentElement!.offsetHeight;
		let t = 0, active = false; const embers: { x: number; y: number; vx: number; vy: number; size: number; life: number; hue: number }[] = [], cracks: { pts: { x: number; y: number }[]; glow: number; width: number }[] = [];
		for (let i = 0; i < 150; i++) embers.push({ x: Math.random() * w, y: h + Math.random() * 100, vx: (Math.random() - 0.5) * 1, vy: -Math.random() * 3 - 1, size: Math.random() * 4 + 1, life: Math.random(), hue: Math.random() * 40 + 10 });
		for (let i = 0; i < 8; i++) { const pts: { x: number; y: number }[] = []; let cx = Math.random() * w, cy = h; for (let j = 0; j < 10; j++) { cx += (Math.random() - 0.5) * 80; cy -= Math.random() * 60 + 20; pts.push({ x: cx, y: cy }); } cracks.push({ pts, glow: 0, width: Math.random() * 3 + 1 }); }
		function resize() { w = c9.width = c9.parentElement!.offsetWidth; h = c9.height = c9.parentElement!.offsetHeight; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; x.fillStyle = 'rgba(5,2,0,0.06)'; x.fillRect(0, 0, w, h);
			const grd = x.createLinearGradient(0, h * 0.7, 0, h); grd.addColorStop(0, 'transparent'); grd.addColorStop(0.5, `rgba(255,60,0,${0.03 + Math.sin(t * 2) * 0.01})`); grd.addColorStop(1, `rgba(255,30,0,${0.06 + Math.sin(t * 3) * 0.02})`); x.fillStyle = grd; x.fillRect(0, h * 0.7, w, h * 0.3);
			cracks.forEach(cr => { cr.glow = 0.3 + Math.sin(t * 2 + cr.pts[0].x * 0.01) * 0.2; x.beginPath(); cr.pts.forEach((p, i) => { const jx = p.x + Math.sin(t * 3 + i) * 3; if (i === 0) x.moveTo(jx, p.y); else x.lineTo(jx, p.y); }); x.strokeStyle = `rgba(255,${80 + Math.floor(cr.glow * 100)},0,${cr.glow})`; x.lineWidth = cr.width; x.shadowBlur = 20; x.shadowColor = `rgba(255,60,0,${cr.glow * 0.6})`; x.stroke(); x.strokeStyle = `rgba(255,200,50,${cr.glow * 0.5})`; x.lineWidth = cr.width * 0.4; x.stroke(); x.shadowBlur = 0; });
			embers.forEach(e => { e.x += e.vx + Math.sin(t * 2 + e.x * 0.01) * 0.5; e.y += e.vy; e.life -= 0.003; if (e.life <= 0 || e.y < -20) { e.x = Math.random() * w; e.y = h + 20; e.life = 1; e.vx = (Math.random() - 0.5) * 1; e.vy = -Math.random() * 3 - 1; } const flickr = 0.5 + Math.sin(t * 20 + e.x) * 0.5; x.beginPath(); x.arc(e.x, e.y, e.size * e.life, 0, Math.PI * 2); x.fillStyle = `hsla(${e.hue},100%,${50 + flickr * 20}%,${e.life * 0.7 * flickr})`; x.shadowBlur = e.size * 3; x.shadowColor = `hsla(${e.hue},100%,50%,${e.life * 0.4})`; x.fill(); x.shadowBlur = 0; });
			const mGrd = x.createRadialGradient(mx, my, 0, mx, my, 100); mGrd.addColorStop(0, 'rgba(255,100,0,0.04)'); mGrd.addColorStop(1, 'transparent'); x.fillStyle = mGrd; x.fillRect(mx - 100, my - 100, 200, 200);
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s9', start: 'top 80%', onEnter: () => { active = true; const tl = gsap.timeline({ delay: 0.5 }); tl.to('#s9t', { opacity: 1, duration: 0.6 }, 0.5); tl.add(() => shake(document.getElementById('s9-content')!, 15, 0.4), 0.5); tl.to('#s9s', { opacity: 1, duration: 1 }, 1.2); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initPredator(); initTimewarp(); initVolcanic(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero h-predator" id="s7">
	<canvas bind:this={c7}></canvas>
	<div class="vignette s7-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s7-content">
		<h1 class="t1" id="s7t">LOCKED</h1>
		<p class="t2" id="s7s">TARGET ACQUIRED — EXECUTING</p>
	</div>
	<span class="label">07 — PREDATOR</span>
	<span class="num">07</span>
</section>
<div class="divider"></div>

<section class="hero h-timewarp" id="s8">
	<canvas bind:this={c8}></canvas>
	<div class="vignette s8-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s8-content">
		<h1 class="t1" id="s8t">WARP</h1>
		<p class="t2" id="s8s">MOVE BEFORE TIME CATCHES UP</p>
	</div>
	<span class="label">08 — WARP</span>
	<span class="num">08</span>
</section>
<div class="divider"></div>

<section class="hero h-volcanic" id="s9">
	<canvas bind:this={c9}></canvas>
	<div class="vignette s9-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s9-content">
		<h1 class="t1" id="s9t">ERUPTION</h1>
		<p class="t2" id="s9s">PRESSURE BUILDS. THEN EXPLODES.</p>
	</div>
	<span class="label">09 — VOLCANIC</span>
	<span class="num">09</span>
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
	.h-predator { background: #020200; }
	.h-timewarp { background: #02000a; }
	.h-volcanic { background: #050200; }
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(4rem, 14vw, 12rem); line-height: 0.85; letter-spacing: 0.02em; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.5rem, 1.2vw, 0.75rem); letter-spacing: 0.6em; margin-top: 20px; opacity: 0; }
	.divider { height: 5vh; background: #000; }
	.s7-vignette { background: radial-gradient(ellipse, transparent 35%, rgba(2,2,0,0.8) 100%); }
	.s8-vignette { background: radial-gradient(ellipse, transparent 30%, rgba(2,0,10,0.8) 100%); }
	.s9-vignette { background: radial-gradient(ellipse at 50% 80%, transparent 20%, rgba(5,2,0,0.8) 100%); }
	#s7t { color: #ffcc00; text-shadow: 0 0 60px rgba(255,200,0,0.3); }
	#s7s { color: rgba(255,200,0,0.4); }
	#s8t { color: #cc66ff; text-shadow: 0 0 60px rgba(200,100,255,0.4); }
	#s8s { color: rgba(200,100,255,0.4); }
	#s9t { background: linear-gradient(180deg, #ffcc00, #ff4400, #cc0000); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#s9s { color: rgba(255,120,0,0.5); }
	.h-predator .label { color: #ffcc00; }
	.h-predator .num { color: #ffcc00; }
	.h-timewarp .label { color: #cc66ff; }
	.h-timewarp .num { color: #cc66ff; }
	.h-volcanic .label { color: #ff6600; }
	.h-volcanic .num { color: #ff4400; }
</style>
