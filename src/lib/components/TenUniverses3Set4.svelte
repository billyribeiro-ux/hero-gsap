<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let c10: HTMLCanvasElement;
	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }

	function initCyberpunk() {
		const x = c10.getContext('2d')!; let w = c10.width = c10.parentElement!.offsetWidth, h = c10.height = c10.parentElement!.offsetHeight;
		let t = 0, active = false; const neonDrops: { x: number; y: number; speed: number; len: number; hue: number; alpha: number }[] = [], arcs: { pts: { x: number; y: number }[]; life: number; hue: number }[] = [];
		const columns = Math.ceil(w / 8);
		for (let i = 0; i < columns; i++) neonDrops.push({ x: i * 8, y: Math.random() * -h, speed: Math.random() * 6 + 3, len: Math.random() * 100 + 50, hue: Math.random() > 0.5 ? 300 : 180, alpha: Math.random() * 0.3 + 0.1 });
		function spawnArc() { if (!active) return; const y = Math.random() * h, pts: { x: number; y: number }[] = []; let px = 0; while (px < w) { pts.push({ x: px, y: y + (Math.random() - 0.5) * 40 }); px += Math.random() * 30 + 10; } arcs.push({ pts, life: 1, hue: Math.random() > 0.5 ? 300 : 180 }); timeouts.push(window.setTimeout(spawnArc, Math.random() * 2000 + 500)); }
		function resize() { w = c10.width = c10.parentElement!.offsetWidth; h = c10.height = c10.parentElement!.offsetHeight; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.01; x.fillStyle = 'rgba(2,0,10,0.12)'; x.fillRect(0, 0, w, h);
			neonDrops.forEach(d => { d.y += d.speed; if (d.y > h + d.len) { d.y = -d.len; d.x = Math.random() * w; } const grd = x.createLinearGradient(d.x, d.y - d.len, d.x, d.y); grd.addColorStop(0, 'transparent'); grd.addColorStop(0.8, `hsla(${d.hue},100%,60%,${d.alpha})`); grd.addColorStop(1, `hsla(${d.hue},100%,80%,${d.alpha * 1.5})`); x.strokeStyle = grd; x.lineWidth = 1.5; x.beginPath(); x.moveTo(d.x, d.y - d.len); x.lineTo(d.x, d.y); x.stroke(); x.beginPath(); x.arc(d.x, d.y, 2, 0, Math.PI * 2); x.fillStyle = `hsla(${d.hue},100%,80%,${d.alpha})`; x.shadowBlur = 10; x.shadowColor = `hsla(${d.hue},100%,50%,0.5)`; x.fill(); x.shadowBlur = 0; });
			arcs.forEach(a => { a.life *= 0.93; x.beginPath(); a.pts.forEach((p, i) => { const jy = p.y + (Math.random() - 0.5) * 8 * a.life; if (i === 0) x.moveTo(p.x, jy); else x.lineTo(p.x, jy); }); x.strokeStyle = `hsla(${a.hue},100%,70%,${a.life * 0.5})`; x.lineWidth = 1 + a.life * 2; x.shadowBlur = 15; x.shadowColor = `hsla(${a.hue},100%,50%,${a.life * 0.5})`; x.stroke(); x.shadowBlur = 0; }); arcs.filter(a => a.life > 0.05);
			const scanY = (t * 200) % h; x.fillStyle = 'rgba(255,0,255,0.03)'; x.fillRect(0, scanY - 2, w, 4);
			const chrGrd = x.createLinearGradient(0, h * 0.85, 0, h); chrGrd.addColorStop(0, 'transparent'); chrGrd.addColorStop(1, 'rgba(100,0,200,0.03)'); x.fillStyle = chrGrd; x.fillRect(0, h * 0.85, w, h * 0.15);
			const mGrd = x.createRadialGradient(mx, my, 0, mx, my, 120); const mHue = (t * 50) % 360; mGrd.addColorStop(0, `hsla(${mHue},100%,60%,0.05)`); mGrd.addColorStop(1, 'transparent'); x.fillStyle = mGrd; x.beginPath(); x.arc(mx, my, 120, 0, Math.PI * 2); x.fill();
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s10', start: 'top 80%', onEnter: () => { active = true; spawnArc(); const tl = gsap.timeline({ delay: 0.3 }); tl.to('#s10t', { opacity: 1, duration: 0.5 }, 0.8); tl.to('#s10s', { opacity: 1, duration: 1 }, 1.5); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initCyberpunk(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero h-cyber" id="s10">
	<canvas bind:this={c10}></canvas>
	<div class="vignette s10-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s10-content">
		<h1 class="t1" id="s10t">NEON</h1>
		<p class="t2" id="s10s">THE FUTURE TRADES IN LIGHT</p>
	</div>
	<span class="label">10 — CYBERPUNK</span>
	<span class="num">10</span>
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
	.h-cyber { background: #02000a; }
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(4rem, 14vw, 12rem); line-height: 0.85; letter-spacing: 0.02em; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.5rem, 1.2vw, 0.75rem); letter-spacing: 0.6em; margin-top: 20px; opacity: 0; }
	.s10-vignette { background: radial-gradient(ellipse, transparent 35%, rgba(2,0,10,0.8) 100%); }
	#s10t { background: linear-gradient(90deg, #ff00ff, #00ffff); -webkit-background-clip: text; background-clip: text; color: transparent; text-shadow: none; }
	#s10s { color: rgba(255,0,255,0.5); }
	.h-cyber .label { color: #ff00ff; }
	.h-cyber .num { color: #00ffff; }
</style>
