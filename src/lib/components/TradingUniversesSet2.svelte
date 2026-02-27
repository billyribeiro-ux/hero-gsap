<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0, prevMx = 0, prevMy = 0, mvx = 0, mvy = 0;
	let h6c1: HTMLCanvasElement, h6c2: HTMLCanvasElement;
	let h7c1: HTMLCanvasElement, h7c2: HTMLCanvasElement;
	let h8c1: HTMLCanvasElement, h8c2: HTMLCanvasElement;
	let h9c1: HTMLCanvasElement, h9c2: HTMLCanvasElement;
	let h10c1: HTMLCanvasElement, h10c2: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }
	function updateMouseVelocity() { mvx = mx - prevMx; mvy = my - prevMy; prevMx = mx; prevMy = my; rafIds.push(requestAnimationFrame(updateMouseVelocity)); }
	function shake(el: HTMLElement, intensity = 20, dur = 0.5) { const s = Math.floor(dur / 0.02); for (let j = 0; j < s; j++) { const f = 1 - j / s; gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.7, delay: j * 0.02 }); } gsap.set(el, { x: 0, y: 0, delay: s * 0.02 }); }

	function initGamma() {
		const x1 = h6c1.getContext('2d')!, x2 = h6c2.getContext('2d')!; let w = h6c1.width = h6c1.parentElement!.offsetWidth, h = h6c1.height = h6c1.parentElement!.offsetHeight;
		let t = 0, active = false, exploded = false; const chains: { x: number; y: number; links: number; active: boolean; hue: number }[] = [];
		for (let i = 0; i < 15; i++) chains.push({ x: Math.random() * w, y: Math.random() * h, links: 0, active: false, hue: Math.random() * 60 + 300 });
		function resize() { [h6c1, h6c2].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h6c1.width; h = h6c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.016; x1.fillStyle = 'rgba(10,0,5,0.05)'; x1.fillRect(0, 0, w, h);
			chains.forEach((c, i) => { if (!c.active && Math.random() > 0.98) { c.active = true; c.links = 1; } if (c.active && Math.random() > 0.95 && c.links < 10) c.links++; const grd = x1.createLinearGradient(c.x, c.y - c.links * 20, c.x, c.y); grd.addColorStop(0, 'transparent'); grd.addColorStop(0.5, `hsla(${c.hue},100%,50%,${0.3 + c.links * 0.05})`); grd.addColorStop(1, `hsla(${c.hue},100%,50%,0.6)`); x1.strokeStyle = grd; x1.lineWidth = 3; x1.lineCap = 'round'; x1.beginPath(); x1.moveTo(c.x, c.y); for (let j = 1; j <= c.links; j++) x1.lineTo(c.x + Math.sin(t * 2 + i + j) * 10, c.y - j * 20); x1.stroke(); });
			x2.clearRect(0, 0, w, h); if (exploded) { for (let i = 0; i < 30; i++) { x2.fillStyle = `rgba(255,${Math.floor(Math.random() * 100)},200,${Math.random() * 0.5})`; x2.fillRect(Math.random() * w, Math.random() * h, Math.random() * 4 + 1, Math.random() * 4 + 1); } } x2.fillStyle = 'rgba(255,0,200,0.2)'; x2.font = 'bold 48px "JetBrains Mono"'; x2.textAlign = 'center'; x2.fillText('Γ', w / 2, h / 2); x2.textAlign = 'start';
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h6', start: 'top 70%', onEnter: () => { active = true; const tl = gsap.timeline({ delay: 0.3 }); tl.to('#h6f', { opacity: 0.8, duration: 0.1 }, 0.5); tl.to('#h6f', { opacity: 0, duration: 1.5 }, 0.6); tl.add(() => { exploded = true; }, 0.5); tl.to('#h6t', { opacity: 1, duration: 0.6, ease: 'power4.out' }, 0.8); tl.add(() => shake(document.getElementById('h6-content')!, 35, 0.6), 0.8); tl.to('#h6s', { opacity: 1, duration: 0.8 }, 1.5); tl.to('#h6s2', { opacity: 1, duration: 0.8 }, 1.9); } }); triggers.push(trigger);
	}

	function initTape() {
		const x1 = h7c1.getContext('2d')!, x2 = h7c2.getContext('2d')!; let w = h7c1.width = h7c1.parentElement!.offsetWidth, h = h7c1.height = h7c1.parentElement!.offsetHeight;
		let t = 0, active = false; const prints: { price: string; size: string; side: 'buy' | 'sell'; y: number; alpha: number }[] = [];
		function addPrint() { if (!active) return; const price = (150 + Math.random() * 50).toFixed(2), size = Math.floor(Math.random() * 1000 + 100).toString(), side = Math.random() > 0.5 ? 'buy' : 'sell'; prints.push({ price, size, side, y: -20, alpha: 1 }); timeouts.push(window.setTimeout(addPrint, Math.random() * 200 + 50)); }
		function resize() { [h7c1, h7c2].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h7c1.width; h = h7c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.016; x1.fillStyle = 'rgba(0,0,0,0.05)'; x1.fillRect(0, 0, w, h);
			prints.forEach((p, i) => { p.y += 2; p.alpha -= 0.005; x1.fillStyle = p.side === 'buy' ? `rgba(0,255,136,${p.alpha})` : `rgba(255,51,68,${p.alpha})`; x1.font = '12px "JetBrains Mono"'; x1.fillText(`${p.price} x ${p.size}`, w * 0.15, p.y); }); prints.filter(p => p.alpha > 0);
			x2.clearRect(0, 0, w, h); const tapeHeight = 60; x2.fillStyle = 'rgba(0,255,136,0.1)'; x2.fillRect(0, h - tapeHeight, w * 0.3, tapeHeight); x2.fillStyle = 'rgba(255,51,68,0.1)'; x2.fillRect(w * 0.7, h - tapeHeight, w * 0.3, tapeHeight); x2.strokeStyle = 'rgba(255,255,255,0.1)'; x2.lineWidth = 1; x2.beginPath(); x2.moveTo(w * 0.5, 0); x2.lineTo(w * 0.5, h); x2.stroke();
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h7', start: 'top 70%', onEnter: () => { active = true; addPrint(); const tl = gsap.timeline({ delay: 0.5 }); tl.to('#h7t', { opacity: 1, duration: 0.8, ease: 'power4.out' }, 0.5); tl.add(() => shake(document.getElementById('h7-content')!, 20, 0.3), 0.5); tl.to('#h7s', { opacity: 1, duration: 0.8 }, 1.3); } }); triggers.push(trigger);
	}

	function initFibonacci() {
		const x1 = h8c1.getContext('2d')!, x2 = h8c2.getContext('2d')!; let w = h8c1.width = h8c1.parentElement!.offsetWidth, h = h8c1.height = h8c1.parentElement!.offsetHeight;
		let t = 0, active = false; const spiral: { a: number; r: number }[] = []; let golden = (1 + Math.sqrt(5)) / 2;
		for (let i = 0; i < 500; i++) { const a = i * golden; const r = 5 + i * 2; spiral.push({ a, r }); }
		function resize() { [h8c1, h8c2].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h8c1.width; h = h8c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; x1.fillStyle = 'rgba(2,0,8,0.03)'; x1.fillRect(0, 0, w, h);
			const cx = w / 2, cy = h / 2;
			x1.strokeStyle = `rgba(255,215,0,${0.1 + Math.sin(t) * 0.05})`; x1.lineWidth = 1; x1.beginPath();
			spiral.forEach((p, i) => { const px = cx + Math.cos(p.a + t * 0.2) * p.r, py = cy + Math.sin(p.a + t * 0.2) * p.r; if (i === 0) x1.moveTo(px, py); else x1.lineTo(px, py); }); x1.stroke();
			[0.236, 0.382, 0.5, 0.618, 0.786].forEach((lvl, i) => { x1.strokeStyle = `rgba(255,215,0,${0.05 + i * 0.02})`; x1.beginPath(); x1.moveTo(0, cy - lvl * h * 0.3); x1.lineTo(w, cy - lvl * h * 0.3); x1.moveTo(0, cy + lvl * h * 0.3); x1.lineTo(w, cy + lvl * h * 0.3); x1.stroke(); });
			x2.clearRect(0, 0, w, h); x2.fillStyle = 'rgba(255,215,0,0.15)'; x2.font = 'bold 72px "JetBrains Mono"'; x2.textAlign = 'center'; x2.fillText('φ', w / 2, h / 2); x2.textAlign = 'start';
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h8', start: 'top 70%', onEnter: () => { active = true; const tl = gsap.timeline({ delay: 0.5 }); tl.to('#h8t', { opacity: 1, duration: 0.8, ease: 'power4.out' }, 0.5); tl.add(() => shake(document.getElementById('h8-content')!, 15, 0.3), 0.5); tl.to('#h8s', { opacity: 1, duration: 0.8 }, 1.3); tl.to('#h8s2', { opacity: 1, duration: 0.8 }, 1.7); } }); triggers.push(trigger);
	}

	function initVolatility() {
		const x1 = h9c1.getContext('2d')!, x2 = h9c2.getContext('2d')!; let w = h9c1.width = h9c1.parentElement!.offsetWidth, h = h9c1.height = h9c1.parentElement!.offsetHeight;
		let t = 0, active = false, flash = false; const lightning: { x: number; y: number; segments: number; alpha: number }[] = [];
		function strike() { if (!active) return; lightning.push({ x: Math.random() * w, y: 0, segments: Math.floor(Math.random() * 5 + 3), alpha: 1 }); flash = true; timeouts.push(window.setTimeout(() => flash = false, 50)); timeouts.push(window.setTimeout(strike, Math.random() * 3000 + 1000)); }
		function resize() { [h9c1, h9c2].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h9c1.width; h = h9c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.016; x1.fillStyle = flash ? 'rgba(50,0,0,0.2)' : 'rgba(3,0,2,0.05)'; x1.fillRect(0, 0, w, h);
			lightning.forEach((l, i) => { l.alpha -= 0.02; x1.strokeStyle = `rgba(255,50,80,${l.alpha})`; x1.lineWidth = 2; x1.beginPath(); x1.moveTo(l.x, l.y); let cx = l.x, cy = l.y; for (let s = 0; s < l.segments; s++) { cx += (Math.random() - 0.5) * 100; cy += h / l.segments; x1.lineTo(cx, cy); } x1.stroke(); }); lightning.filter(l => l.alpha > 0);
			x2.clearRect(0, 0, w, h); x2.fillStyle = 'rgba(255,50,80,0.1)'; x2.font = 'bold 80px "JetBrains Mono"'; x2.textAlign = 'center'; const vix = Math.floor(15 + Math.sin(t) * 10 + Math.random() * 5); x2.fillText(`VIX ${vix}`, w / 2, h / 2); x2.textAlign = 'start';
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h9', start: 'top 70%', onEnter: () => { active = true; strike(); const tl = gsap.timeline({ delay: 0.3 }); tl.to('#h9f', { opacity: 0.4, duration: 0.05 }, 0.4); tl.to('#h9f', { opacity: 0, duration: 2 }, 0.45); tl.to('#h9t', { opacity: 1, duration: 0.8, ease: 'power4.out' }, 0.8); tl.add(() => shake(document.getElementById('h9-content')!, 25, 0.5), 0.8); tl.to('#h9s', { opacity: 1, duration: 0.8 }, 1.5); tl.to('#h9s2', { opacity: 1, duration: 0.8 }, 1.9); } }); triggers.push(trigger);
	}

	function initEdge() {
		const x1 = h10c1.getContext('2d')!, x2 = h10c2.getContext('2d')!; let w = h10c1.width = h10c1.parentElement!.offsetWidth, h = h10c1.height = h10c1.parentElement!.offsetHeight;
		let t = 0, active = false; const cloud: { x: number; y: number; vx: number; vy: number; r: number; prob: number }[] = [];
		for (let i = 0; i < 100; i++) cloud.push({ x: Math.random() * w, y: Math.random() * h, vx: (Math.random() - 0.5) * 0.5, vy: (Math.random() - 0.5) * 0.5, r: Math.random() * 80 + 40, prob: Math.random() });
		function resize() { [h10c1, h10c2].forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = h10c1.width; h = h10c1.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!active) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; x1.fillStyle = 'rgba(0,2,8,0.03)'; x1.fillRect(0, 0, w, h);
			cloud.forEach(p => { p.x += p.vx; p.y += p.vy; if (p.x < -100) p.x = w + 100; if (p.x > w + 100) p.x = -100; if (p.y < -100) p.y = h + 100; if (p.y > h + 100) p.y = -100; const grd = x1.createRadialGradient(p.x, p.y, 0, p.x, p.y, p.r); const hue = p.prob > 0.55 ? 200 : 220; const alpha = 0.05 + Math.abs(p.prob - 0.5) * 0.1; grd.addColorStop(0, `hsla(${hue},80%,60%,${alpha})`); grd.addColorStop(1, 'transparent'); x1.fillStyle = grd; x1.fillRect(p.x - p.r, p.y - p.r, p.r * 2, p.r * 2); });
			x2.clearRect(0, 0, w, h); x2.fillStyle = 'rgba(200,230,255,0.1)'; x2.font = 'bold 36px "JetBrains Mono"'; x2.textAlign = 'center'; x2.fillText('EDGE: +2.3σ', w / 2, h / 2); x2.textAlign = 'start';
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#h10', start: 'top 70%', onEnter: () => { active = true; const tl = gsap.timeline({ delay: 0.5 }); tl.to('#h10t', { opacity: 1, duration: 0.8, ease: 'power4.out' }, 0.5); tl.add(() => shake(document.getElementById('h10-content')!, 15, 0.3), 0.5); tl.to('#h10s', { opacity: 1, duration: 0.8 }, 1.3); tl.to('#h10s2', { opacity: 1, duration: 0.8 }, 1.7); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; prevMx = mx; prevMy = my; rafIds.push(requestAnimationFrame(updateMouseVelocity)); initGamma(); initTape(); initFibonacci(); initVolatility(); initEdge(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

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
<div class="sp"></div>

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
<div class="sp"></div>

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
	.vig { position: absolute; inset: 0; z-index: 5; pointer-events: none; }
	.grn { position: absolute; inset: 0; z-index: 8; pointer-events: none; opacity: 0.03; background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 400 400' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E"); }
	.flash { position: absolute; inset: 0; z-index: 15; pointer-events: none; opacity: 0; }
	.ct { position: relative; z-index: 20; text-align: center; pointer-events: none; }
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(5rem, 12vw, 10rem); line-height: 0.85; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: 0.65rem; letter-spacing: 0.5em; opacity: 0; margin-top: 15px; }
	.t3 { font-family: 'JetBrains Mono', monospace; font-size: 0.5rem; letter-spacing: 0.3em; opacity: 0; margin-top: 25px; color: rgba(255,255,255,0.3); }
	.lbl { position: absolute; top: 25px; left: 30px; z-index: 25; font-size: 0.6rem; letter-spacing: 0.4em; opacity: 0.4; }
	.idx { position: absolute; bottom: 30px; right: 35px; z-index: 25; font-family: 'Bebas Neue', sans-serif; font-size: 8rem; opacity: 0.05; }
	.sp { height: 8vh; background: #000; }

	.h6-vig { background: radial-gradient(ellipse, transparent 30%, rgba(10,0,5,0.85) 100%); }
	.h7-vig { background: radial-gradient(ellipse, transparent 40%, rgba(0,0,0,0.7) 100%); }
	.h8-vig { background: radial-gradient(ellipse, transparent 30%, rgba(2,0,8,0.85) 100%); }
	.h9-vig { background: radial-gradient(ellipse at 50% 40%, transparent 20%, rgba(3,0,2,0.9) 100%); }
	.h10-vig { background: radial-gradient(ellipse, transparent 35%, rgba(0,2,8,0.85) 100%); }

	#h6t { background: linear-gradient(90deg, #ff00cc, #ff6600); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#h6s { color: rgba(255,0,200,0.5); }
	#h6s2 { color: rgba(255,100,0,0.3); }
	#h7t { color: #fff; text-shadow: 0 0 40px rgba(0,255,136,0.2); }
	#h7s { color: rgba(0,255,136,0.4); }
	#h8t { background: linear-gradient(180deg, #ffd700, #ff8c00, #cc6600); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#h8s { color: rgba(255,215,0,0.4); }
	#h8s2 { color: rgba(255,215,0,0.2); }
	#h9t { color: #ff2244; text-shadow: 0 0 100px rgba(255,34,68,0.5); }
	#h9s { color: rgba(255,34,68,0.4); }
	#h9s2 { color: rgba(255,34,68,0.2); }
	#h10t { background: linear-gradient(180deg, #fff, #00ddff, #0066ff); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#h10s { color: rgba(0,200,255,0.4); }
	#h10s2 { color: rgba(0,200,255,0.2); }

	.h6-lbl { color: #ff00cc; }
	.h6-idx { color: #ff6600; }
	.h7-lbl { color: #00ff88; }
	.h7-idx { color: #00ff88; }
	.h8-lbl { color: #ffd700; }
	.h8-idx { color: #ff8c00; }
	.h9-lbl { color: #ff2244; }
	.h9-idx { color: #ff2244; }
	.h10-lbl { color: #00ddff; }
	.h10-idx { color: #00ddff; }
</style>
