<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let h7c1: HTMLCanvasElement, h7c2: HTMLCanvasElement;
	let h8c1: HTMLCanvasElement, h8c2: HTMLCanvasElement;
	let h9c1: HTMLCanvasElement, h9c2: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }
	function shake(el: HTMLElement, intensity = 20, dur = 0.5) { const s = Math.floor(dur / 0.02); for (let j = 0; j < s; j++) { const f = 1 - j / s; gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.7, delay: j * 0.02 }); } gsap.set(el, { x: 0, y: 0, delay: s * 0.02 }); }

	function initTape() {
		const x1 = h7c1.getContext('2d')!, x2 = h7c2.getContext('2d')!; let w = h7c1.width = h7c1.parentElement!.offsetWidth, h = h7c1.height = h7c1.parentElement!.offsetHeight;
		let t = 0, active = false; const tapeEntries: { col: number; price: string; size: number; side: boolean; y: number; speed: number; alpha: number; isBlock: boolean; glow: number }[] = [], waves: { y: number; alpha: number; x: number }[] = [];
		function addTape() { if (!active) return; const isBlock = Math.random() > 0.92; const size = isBlock ? Math.floor(Math.random() * 5000 + 1000) : Math.floor(Math.random() * 50 + 1); const price = (5890 + Math.random() * 20 - 10).toFixed(2); const side = Math.random() > 0.5; const col = Math.floor(Math.random() * 5); tapeEntries.push({ col, price, size, side, y: -20, speed: 2 + Math.random() * 3, alpha: isBlock ? 0.8 : 0.3, isBlock, glow: isBlock ? 1 : 0 }); if (isBlock) waves.push({ y: 0, alpha: 0.3, x: col * (w / 5) + w / 10 }); timeouts.push(window.setTimeout(addTape, isBlock ? 500 : 30 + Math.random() * 50)); }
		function resize() { [h7c1, h7c2].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h7c1.width; h = h7c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.01; x1.fillStyle = 'rgba(0,0,0,0.08)'; x1.fillRect(0, 0, w, h); x2.clearRect(0, 0, w, h);
			for (let i = 1; i < 5; i++) { x2.strokeStyle = 'rgba(0,255,136,0.03)'; x2.lineWidth = 0.5; x2.beginPath(); x2.moveTo(i * w / 5, 0); x2.lineTo(i * w / 5, h); x2.stroke(); }
			tapeEntries.forEach(e => { e.y += e.speed; const ex = e.col * (w / 5) + 10; const color = e.side ? '0,255,136' : '255,51,68'; const a = e.alpha * (1 - e.y / h * 0.5); if (e.isBlock) { x1.fillStyle = `rgba(${color},${a * 0.08})`; x1.fillRect(ex - 5, e.y - 8, w / 5 - 10, 18); e.glow *= 0.99; } x2.font = `${e.isBlock ? 'bold 11' : '10'}px "JetBrains Mono"`; x2.fillStyle = `rgba(${color},${a})`; x2.fillText(`${e.price}`, ex, e.y); x2.fillText(`${e.size}`, ex + 80, e.y); if (e.isBlock) { x2.shadowBlur = 10; x2.shadowColor = `rgba(${color},0.5)`; x2.fillText('■ BLOCK', ex + 140, e.y); x2.shadowBlur = 0; } }); tapeEntries.filter(e => e.y < h + 50);
			waves.forEach(wv => { wv.alpha *= 0.97; x2.strokeStyle = `rgba(255,255,255,${wv.alpha})`; x2.lineWidth = 1; x2.beginPath(); x2.moveTo(0, wv.y); x2.lineTo(w, wv.y); x2.stroke(); wv.y += 5; }); waves.filter(wv => wv.alpha > 0.01);
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h7', start: 'top 70%', onEnter: () => { active = true; addTape(); const tl = gsap.timeline({ delay: 0.5 }); tl.to('#h7t', { opacity: 1, duration: 0.5 }, 1); tl.to('#h7s', { opacity: 1, duration: 1 }, 1.5); } }); triggers.push(trigger);
	}

	function initFibonacci() {
		const x1 = h8c1.getContext('2d')!, x2 = h8c2.getContext('2d')!; let w = h8c1.width = h8c1.parentElement!.offsetWidth, h = h8c1.height = h8c1.parentElement!.offsetHeight;
		let t = 0, active = false, spiralProgress = 0; const PHI = 1.618033988749, LEVELS = [0.236, 0.382, 0.5, 0.618, 0.786];
		function resize() { [h8c1, h8c2].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h8c1.width; h = h8c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.005; x1.fillStyle = 'rgba(2,0,8,0.03)'; x1.fillRect(0, 0, w, h); x2.clearRect(0, 0, w, h);
			const cx = w * 0.45 + (mx / w - 0.5) * 30, cy = h * 0.55 + (my / h - 0.5) * 30;
			if (spiralProgress > 0) { x1.beginPath(); let r = 5; const maxAngle = spiralProgress * Math.PI * 8; for (let a = 0; a < maxAngle; a += 0.02) { r = 5 * Math.pow(PHI, a / (Math.PI * 2)); const px = cx + Math.cos(a + t * 0.3) * r, py = cy + Math.sin(a + t * 0.3) * r; if (a === 0) x1.moveTo(px, py); else x1.lineTo(px, py); } x1.strokeStyle = `rgba(255,215,0,${0.15 + Math.sin(t * 3) * 0.05})`; x1.lineWidth = 1.5; x1.shadowBlur = 10; x1.shadowColor = 'rgba(255,215,0,0.3)'; x1.stroke(); x1.shadowBlur = 0; let rw = 30 * spiralProgress, rh = rw / PHI; for (let i = 0; i < 6 && i < spiralProgress * 8; i++) { x2.save(); x2.translate(cx, cy); x2.rotate(i * Math.PI / 2 + t * 0.1); x2.strokeStyle = `rgba(255,215,0,${0.04 * (6 - i) / 6})`; x2.lineWidth = 0.5; x2.strokeRect(-rw / 2, -rh / 2, rw, rh); x2.restore(); const tmp = rw; rw = rw + rh; rh = tmp; } }
			LEVELS.forEach((lv, i) => { const ly = h * 0.15 + lv * h * 0.7; const alpha = 0.05 + Math.sin(t * 2 + i) * 0.02; x2.setLineDash([2, 6]); x2.strokeStyle = `rgba(255,215,0,${alpha})`; x2.lineWidth = 0.5; x2.beginPath(); x2.moveTo(w * 0.1, ly); x2.lineTo(w * 0.9, ly); x2.stroke(); x2.setLineDash([]); x2.fillStyle = `rgba(255,215,0,${alpha * 2})`; x2.font = '8px "JetBrains Mono"'; x2.fillText(lv.toFixed(3), w * 0.91, ly + 3); const dist = Math.abs(my - ly); if (dist < 40) { const grd = x2.createLinearGradient(0, ly - 20, 0, ly + 20); grd.addColorStop(0, 'transparent'); grd.addColorStop(0.5, `rgba(255,215,0,${(1 - dist / 40) * 0.08})`); grd.addColorStop(1, 'transparent'); x2.fillStyle = grd; x2.fillRect(0, ly - 20, w, 40); } });
			const ratios = ['1.618', '0.618', '2.618', '0.382', '1.000']; ratios.forEach((r, i) => { const rx = w * 0.1 + i * w * 0.2 + Math.sin(t * 0.5 + i) * 20; const ry = h * 0.9 + Math.cos(t * 0.7 + i) * 10; x2.fillStyle = `rgba(255,215,0,${0.05 + Math.sin(t + i) * 0.02})`; x2.font = '18px Orbitron'; x2.fillText(r, rx, ry); });
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h8', start: 'top 70%', onEnter: () => { active = true; gsap.to({ v: 0 }, { v: 1, duration: 4, ease: 'power2.out', onUpdate: function() { spiralProgress = this.targets()[0].v; } }); const tl = gsap.timeline({ delay: 0.5 }); tl.to('#h8t', { opacity: 1, duration: 1, ease: 'power2.out' }, 1.5); tl.to('#h8s', { opacity: 1, duration: 1 }, 2.5); tl.to('#h8s2', { opacity: 1, duration: 0.8 }, 3.2); } }); triggers.push(trigger);
	}

	function initVolatility() {
		const x1 = h9c1.getContext('2d')!, x2 = h9c2.getContext('2d')!; let w = h9c1.width = h9c1.parentElement!.offsetWidth, h = h9c1.height = h9c1.parentElement!.offsetHeight;
		let t = 0, active = false; const clouds: { x: number; y: number; r: number; vx: number; darkness: number }[] = [], bolts: { pts: { x: number; y: number; branch?: boolean; resume?: boolean }[]; life: number }[] = [], rain: { x: number; y: number; speed: number; len: number; alpha: number }[] = [];
		for (let i = 0; i < 20; i++) clouds.push({ x: Math.random() * w, y: Math.random() * h * 0.4, r: Math.random() * 120 + 60, vx: (Math.random() - 0.5) * 0.5, darkness: Math.random() * 0.3 + 0.1 });
		for (let i = 0; i < 400; i++) rain.push({ x: Math.random() * w, y: Math.random() * h, speed: Math.random() * 8 + 4, len: Math.random() * 20 + 10, alpha: Math.random() * 0.2 + 0.05 });
		function lightning() { if (!active) return; const pts: { x: number; y: number; branch?: boolean; resume?: boolean }[] = [{ x: Math.random() * w * 0.6 + w * 0.2, y: 0 }]; let y = 0; while (y < h * 0.7) { y += Math.random() * 40 + 20; const lastX = pts[pts.length - 1].x; pts.push({ x: lastX + (Math.random() - 0.5) * 80, y }); if (Math.random() > 0.6) { let bx = lastX, by = y; for (let i = 0; i < 3; i++) { bx += (Math.random() - 0.5) * 40; by += Math.random() * 30 + 10; pts.push({ x: bx, y: by, branch: true }); } pts.push({ x: lastX + (Math.random() - 0.5) * 80, y: y + 10, resume: true }); } } bolts.push({ pts, life: 1 }); gsap.to('#h9f', { opacity: 0.5, duration: 0.03, onComplete: () => gsap.to('#h9f', { opacity: 0, duration: 0.4 }) }); shake(document.getElementById('h9-content')!, 10, 0.2); timeouts.push(window.setTimeout(lightning, Math.random() * 3000 + 1000)); }
		function resize() { [h9c1, h9c2].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h9c1.width; h = h9c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.006; x1.fillStyle = 'rgba(3,0,2,0.08)'; x1.fillRect(0, 0, w, h); x2.clearRect(0, 0, w, h);
			clouds.forEach(cl => { cl.x += cl.vx; if (cl.x < -cl.r) cl.x = w + cl.r; if (cl.x > w + cl.r) cl.x = -cl.r; const grd = x1.createRadialGradient(cl.x, cl.y, 0, cl.x, cl.y, cl.r); grd.addColorStop(0, `rgba(40,10,20,${cl.darkness})`); grd.addColorStop(1, 'transparent'); x1.fillStyle = grd; x1.fillRect(cl.x - cl.r, cl.y - cl.r, cl.r * 2, cl.r * 2); });
			rain.forEach(r => { r.y += r.speed; r.x -= 1; if (r.y > h) { r.y = -r.len; r.x = Math.random() * w; } x1.beginPath(); x1.moveTo(r.x, r.y); x1.lineTo(r.x - 2, r.y + r.len); x1.strokeStyle = `rgba(180,130,140,${r.alpha})`; x1.lineWidth = 0.5; x1.stroke(); });
			bolts.forEach(b => { b.life *= 0.9; x2.beginPath(); let branching = false; b.pts.forEach((p, i) => { if (p.resume) { branching = false; return; } if (p.branch && !branching) { x2.stroke(); x2.beginPath(); x2.moveTo(p.x, p.y); branching = true; return; } const jx = p.x + (Math.random() - 0.5) * 4 * b.life, jy = p.y + (Math.random() - 0.5) * 2 * b.life; if (i === 0 || p.resume) x2.moveTo(jx, jy); else x2.lineTo(jx, jy); }); x2.strokeStyle = `rgba(200,180,255,${b.life * 0.8})`; x2.lineWidth = 1 + b.life * 3; x2.shadowBlur = 20; x2.shadowColor = `rgba(200,150,255,${b.life})`; x2.stroke(); x2.shadowBlur = 0; }); bolts.filter(b => b.life > 0.05);
			const vix = 18 + Math.sin(t * 0.8) * 12 + Math.sin(t * 3) * 5; const gaugeX = w - 120, gaugeY = h - 80; x2.beginPath(); x2.arc(gaugeX, gaugeY, 50, -Math.PI, 0); x2.strokeStyle = 'rgba(255,34,68,0.15)'; x2.lineWidth = 4; x2.stroke(); const vixAngle = -Math.PI + ((vix - 5) / 50) * Math.PI; x2.beginPath(); x2.moveTo(gaugeX, gaugeY); x2.lineTo(gaugeX + Math.cos(vixAngle) * 45, gaugeY + Math.sin(vixAngle) * 45); x2.strokeStyle = 'rgba(255,34,68,0.6)'; x2.lineWidth = 2; x2.stroke(); x2.fillStyle = 'rgba(255,34,68,0.4)'; x2.font = 'bold 18px Orbitron'; x2.textAlign = 'center'; x2.fillText(vix.toFixed(1), gaugeX, gaugeY + 25); x2.font = '8px "JetBrains Mono"'; x2.fillText('VIX', gaugeX, gaugeY + 38); x2.textAlign = 'start';
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h9', start: 'top 70%', onEnter: () => { active = true; timeouts.push(window.setTimeout(lightning, 800)); const tl = gsap.timeline({ delay: 0.5 }); tl.to('#h9t', { opacity: 1, duration: 0.4 }, 1); tl.add(() => shake(document.getElementById('h9-content')!, 20, 0.4), 1); tl.to('#h9s', { opacity: 1, duration: 1 }, 1.5); tl.to('#h9s2', { opacity: 1, duration: 0.8 }, 2.2); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initTape(); initFibonacci(); initVolatility(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="h7" style="background:#000">
	<canvas bind:this={h7c1}></canvas>
	<canvas bind:this={h7c2} class="c2"></canvas>
	<div class="vig h7-vig"></div>
	<div class="grn"></div>
	<div class="ct" id="h7-content">
		<h1 class="t1" id="h7t">TAPE</h1>
		<p class="t2" id="h7s">READ THE FLOW. KNOW THE MOVE.</p>
	</div>
	<span class="lbl h7-lbl">07 — TAPE READER</span>
	<span class="idx h7-idx">07</span>
</section>
<div class="sp"></div>

<section class="hero" id="h8" style="background:#020008">
	<canvas bind:this={h8c1}></canvas>
	<canvas bind:this={h8c2} class="c2"></canvas>
	<div class="vig h8-vig"></div>
	<div class="grn"></div>
	<div class="ct" id="h8-content">
		<h1 class="t1" id="h8t">FIBONACCI</h1>
		<p class="t2" id="h8s">THE UNIVERSE TRADES IN RATIOS</p>
		<p class="t3" id="h8s2">[ 0.236 — 0.382 — 0.618 — 0.786 ]</p>
	</div>
	<span class="lbl h8-lbl">08 — FIBONACCI</span>
	<span class="idx h8-idx">08</span>
</section>
<div class="sp"></div>

<section class="hero" id="h9" style="background:#030002">
	<canvas bind:this={h9c1}></canvas>
	<canvas bind:this={h9c2} class="c2"></canvas>
	<div class="vig h9-vig"></div>
	<div class="grn"></div>
	<div class="flash" id="h9f" style="background:#fff"></div>
	<div class="ct" id="h9-content">
		<h1 class="t1" id="h9t">VOLATILITY</h1>
		<p class="t2" id="h9s">FEAR IS THE MOST EXPENSIVE EMOTION</p>
		<p class="t3" id="h9s2">[ VIX: EXTREME ]</p>
	</div>
	<span class="lbl h9-lbl">09 — VOLATILITY</span>
	<span class="idx h9-idx">09</span>
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
	.h7-vig { background: radial-gradient(ellipse, transparent 40%, rgba(0,0,0,0.7) 100%); }
	.h8-vig { background: radial-gradient(ellipse, transparent 30%, rgba(2,0,8,0.85) 100%); }
	.h9-vig { background: radial-gradient(ellipse at 50% 40%, transparent 20%, rgba(3,0,2,0.9) 100%); }
	#h7t { color: #fff; text-shadow: 0 0 40px rgba(0,255,136,0.2); }
	#h7s { color: rgba(0,255,136,0.4); }
	#h8t { background: linear-gradient(180deg, #ffd700, #ff8c00, #cc6600); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#h8s { color: rgba(255,215,0,0.4); }
	#h8s2 { color: rgba(255,215,0,0.2); }
	#h9t { color: #ff2244; text-shadow: 0 0 100px rgba(255,34,68,0.5); }
	#h9s { color: rgba(255,34,68,0.4); }
	#h9s2 { color: rgba(255,34,68,0.2); }
	.h7-lbl, .h7-idx { color: #00ff88; }
	.h8-lbl, .h8-idx { color: #ffd700; }
	.h9-lbl, .h9-idx { color: #ff2244; }
</style>
