<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let h10c1: HTMLCanvasElement, h10c2: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }

	function initEdge() {
		const x1 = h10c1.getContext('2d')!, x2 = h10c2.getContext('2d')!; let w = h10c1.width = h10c1.parentElement!.offsetWidth, h = h10c1.height = h10c1.parentElement!.offsetHeight;
		let t = 0, active = false, evLine = 0; const dataPoints: { x: number; y: number; ox: number; oy: number; vx: number; vy: number; size: number; alpha: number; dist: number }[] = [];
		for (let i = 0; i < 500; i++) { const u1 = Math.random(), u2 = Math.random(), z = Math.sqrt(-2 * Math.log(u1)) * Math.cos(2 * Math.PI * u2), z2 = Math.sqrt(-2 * Math.log(u1)) * Math.sin(2 * Math.PI * u2); dataPoints.push({ x: w / 2 + z * w * 0.15, y: h / 2 + z2 * h * 0.15, ox: w / 2 + z * w * 0.15, oy: h / 2 + z2 * h * 0.15, vx: 0, vy: 0, size: Math.random() * 2.5 + 0.5, alpha: Math.random() * 0.3 + 0.1, dist: Math.sqrt(z * z + z2 * z2) }); }
		function resize() { [h10c1, h10c2].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h10c1.width; h = h10c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.006; x1.fillStyle = 'rgba(0,2,8,0.04)'; x1.fillRect(0, 0, w, h); x2.clearRect(0, 0, w, h);
			const cx = w / 2, cy = h / 2; x2.beginPath(); for (let i = 0; i < w; i += 3) { const z = (i - cx) / (w * 0.15); const bell = Math.exp(-z * z / 2) * h * 0.35; if (i === 0) x2.moveTo(i, cy - bell); else x2.lineTo(i, cy - bell); } x2.strokeStyle = `rgba(0,200,255,${0.06 + Math.sin(t * 2) * 0.02})`; x2.lineWidth = 1; x2.stroke();
			x2.beginPath(); for (let i = 0; i < w; i += 3) { const z = (i - cx) / (w * 0.15); const bell = Math.exp(-z * z / 2) * h * 0.35; if (i === 0) x2.moveTo(i, cy + bell); else x2.lineTo(i, cy + bell); } x2.strokeStyle = 'rgba(0,200,255,0.03)'; x2.stroke();
			[1, 2, 3].forEach(sd => { const xPos = cx + sd * w * 0.15; const xNeg = cx - sd * w * 0.15; x2.strokeStyle = `rgba(0,200,255,${0.04 - sd * 0.01})`; x2.setLineDash([2, 4]); x2.lineWidth = 0.5; [xPos, xNeg].forEach(px => { x2.beginPath(); x2.moveTo(px, cy - h * 0.4); x2.lineTo(px, cy + h * 0.4); x2.stroke(); }); x2.setLineDash([]); x2.fillStyle = 'rgba(0,200,255,0.04)'; x2.font = '7px "JetBrains Mono"'; x2.fillText(`${sd}σ`, xPos + 3, cy - h * 0.35); x2.fillText(`-${sd}σ`, xNeg + 3, cy - h * 0.35); });
			dataPoints.forEach(p => { const dx = mx - p.x, dy = my - p.y; const dist = Math.sqrt(dx * dx + dy * dy); if (dist < 200) { p.vx += (dx / dist) * 0.3 * (1 - dist / 200); p.vy += (dy / dist) * 0.3 * (1 - dist / 200); } p.vx += (p.ox - p.x) * 0.01; p.vy += (p.oy - p.y) * 0.01; p.vx *= 0.95; p.vy *= 0.95; p.x += p.vx; p.y += p.vy; const r = Math.min(255, p.dist * 80); const b = 255 - r; x1.beginPath(); x1.arc(p.x, p.y, p.size, 0, Math.PI * 2); x1.fillStyle = `rgba(${r},${Math.floor(b * 0.7)},${b},${p.alpha})`; if (p.dist < 0.5) { x1.shadowBlur = 4; x1.shadowColor = 'rgba(0,200,255,0.2)'; } x1.fill(); x1.shadowBlur = 0; });
			if (evLine > 0) { x2.strokeStyle = `rgba(0,255,200,${evLine * 0.3})`; x2.lineWidth = 2; x2.setLineDash([]); x2.beginPath(); x2.moveTo(cx, cy - h * 0.4 * evLine); x2.lineTo(cx, cy + h * 0.4 * evLine); x2.stroke(); x2.shadowBlur = 15; x2.shadowColor = 'rgba(0,255,200,0.4)'; x2.stroke(); x2.shadowBlur = 0; x2.fillStyle = `rgba(0,255,200,${evLine * 0.5})`; x2.font = 'bold 10px "JetBrains Mono"'; x2.fillText('E[V] = +', cx + 8, cy - h * 0.35 * evLine); }
			const wr = 0.63 + Math.sin(t * 0.5) * 0.05; x2.fillStyle = 'rgba(0,200,255,0.08)'; x2.font = '40px Orbitron'; x2.textAlign = 'center'; x2.fillText((wr * 100).toFixed(1) + '%', 80, h - 50); x2.font = '8px "JetBrains Mono"'; x2.fillText('WIN RATE', 80, h - 30); x2.textAlign = 'start';
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h10', start: 'top 70%', onEnter: () => { active = true; gsap.to({ v: 0 }, { v: 1, duration: 2, delay: 1, ease: 'power2.out', onUpdate: function() { evLine = this.targets()[0].v; } }); const tl = gsap.timeline({ delay: 0.5 }); tl.to('#h10t', { opacity: 1, duration: 0.8, ease: 'power2.out' }, 1); tl.to('#h10s', { opacity: 1, duration: 1 }, 1.8); tl.to('#h10s2', { opacity: 1, duration: 0.8 }, 2.5); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initEdge(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="h10" style="background:#000208">
	<canvas bind:this={h10c1}></canvas>
	<canvas bind:this={h10c2} class="c2"></canvas>
	<div class="vig h10-vig"></div>
	<div class="grn"></div>
	<div class="ct" id="h10-content">
		<h1 class="t1" id="h10t">THE EDGE</h1>
		<p class="t2" id="h10s">PROBABILITY IS THE ONLY TRUTH</p>
		<p class="t3" id="h10s2">[ EXPECTED VALUE: POSITIVE ]</p>
	</div>
	<span class="lbl h10-lbl">10 — THE EDGE</span>
	<span class="idx h10-idx">10</span>
</section>

<style>
	:global(*) { margin: 0; padding: 0; box-sizing: border-box; }
	:global(html) { overflow-x: hidden; background: #000; }
	.hero { position: relative; width: 100%; height: 100vh; overflow: hidden; display: flex; align-items: center; justify-content: center; color: #fff; font-family: 'JetBrains Mono', monospace; }
	.hero canvas { position: absolute; inset: 0; width: 100%; height: 100%; z-index: 1; }
	.c2 { z-index: 2; }
	.vig { position: absolute; inset: 0; z-index: 40; pointer-events: none; }
	.grn { position: absolute; inset: 0; z-index: 50; pointer-events: none; opacity: 0.03; mix-blend-mode: overlay; background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E"); background-size: 128px; }
	.ct { position: relative; z-index: 10; text-align: center; pointer-events: none; }
	.lbl { position: absolute; top: 25px; left: 30px; z-index: 60; font-size: 0.45rem; letter-spacing: 0.6em; text-transform: uppercase; opacity: 0.15; }
	.idx { position: absolute; bottom: 25px; right: 35px; z-index: 60; font-family: 'Bebas Neue'; font-size: 5rem; line-height: 1; opacity: 0.03; }
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(3.5rem, 13vw, 11rem); line-height: 0.85; letter-spacing: 0.02em; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.45rem, 1.1vw, 0.7rem); letter-spacing: 0.7em; margin-top: 18px; opacity: 0; }
	.t3 { font-family: 'JetBrains Mono', monospace; font-size: 0.5rem; letter-spacing: 0.35em; margin-top: 28px; opacity: 0; }
	.h10-vig { background: radial-gradient(ellipse, transparent 35%, rgba(0,2,8,0.85) 100%); }
	#h10t { background: linear-gradient(180deg, #fff, #00ddff, #0066ff); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#h10s { color: rgba(0,200,255,0.4); }
	#h10s2 { color: rgba(0,200,255,0.2); }
	.h10-lbl, .h10-idx { color: #00ddff; }
</style>
