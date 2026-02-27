<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let s10a: HTMLCanvasElement, s10b: HTMLCanvasElement, s10c: HTMLCanvasElement, s10d: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }
	function shake(el: HTMLElement, intensity = 20, dur = 0.5) { const s = Math.floor(dur / 0.016); for (let i = 0; i < s; i++) { const f = Math.pow(1 - i / s, 2); gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.6, delay: i * 0.016 }); } gsap.set(el, { x: 0, y: 0, delay: s * 0.016 }); }

	function initTheMove() {
		const cv = [s10a, s10b, s10c, s10d];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s10a.width = s10a.parentElement!.offsetWidth, h = s10a.height = s10a.parentElement!.offsetHeight;
		let t = 0, on = false, phase = 'build';
		const candles: { o: number; c: number; h: number; l: number; bull: boolean; visible: boolean; breakout?: boolean }[] = [];
		let price = 5850;
		function genChart() { candles.length = 0; for (let i = 0; i < 30; i++) { const o = price; price += (Math.random() - 0.5) * 6; const c = price; candles.push({ o, c, h: Math.max(o, c) + Math.random() * 3, l: Math.min(o, c) - Math.random() * 3, bull: c > o, visible: false }); } for (let i = 0; i < 12; i++) { const o = price; price += Math.random() * 25 + 8; const c = price; candles.push({ o, c, h: c + Math.random() * 5, l: o - Math.random() * 2, bull: true, visible: false, breakout: true }); } }
		genChart();
		const shards: { x: number; y: number; vx: number; vy: number; size: number; rot: number; rv: number; life: number; color: number[] }[] = [];
		const cloudParts: { x: number; y: number; vx: number; vy: number; size: number; life: number; hue: number }[] = [];
		const shockwaves: { r: number; alpha: number; width: number }[] = [];
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s10a.width; h = s10a.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; const [xa, xb, xc, xd] = ctx;
			xa.fillStyle = `rgba(0,0,0,${phase === 'boom' ? 0.06 : 0.1})`; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xc.clearRect(0, 0, w, h); xd.clearRect(0, 0, w, h);
			if (phase === 'build') {
				const vis = candles.filter(c => c.visible); if (vis.length === 0) { rafIds.push(requestAnimationFrame(draw)); return; }
				const prices = vis.flatMap(c => [c.h, c.l]), minP = Math.min(...prices) - 5, maxP = Math.max(...prices) + 5, range = maxP - minP, gap = w * 0.65 / candles.length, cw = Math.min(gap * 0.65, 11), offX = w * 0.17;
				vis.forEach((c, idx) => { const i = candles.indexOf(c), x = offX + i * gap + gap / 2, oY = h * 0.12 + (maxP - c.o) / range * h * 0.6, cY = h * 0.12 + (maxP - c.c) / range * h * 0.6, hY = h * 0.12 + (maxP - c.h) / range * h * 0.6, lY = h * 0.12 + (maxP - c.l) / range * h * 0.6; xb.strokeStyle = c.bull ? 'rgba(0,255,136,0.6)' : 'rgba(255,51,68,0.6)'; xb.lineWidth = 1.5; xb.beginPath(); xb.moveTo(x, hY); xb.lineTo(x, lY); xb.stroke(); xb.fillStyle = c.bull ? 'rgba(0,255,136,0.85)' : 'rgba(255,51,68,0.85)'; xb.fillRect(x - cw / 2, Math.min(oY, cY), cw, Math.abs(cY - oY) || 1.5); if (c.breakout) { xb.shadowBlur = 25; xb.shadowColor = 'rgba(0,255,136,0.7)'; xb.fillRect(x - cw / 2, Math.min(oY, cY), cw, Math.abs(cY - oY) || 1.5); xb.shadowBlur = 0; } });
				const lastVis = vis[vis.length - 1]; if (lastVis) { xb.fillStyle = 'rgba(0,255,136,0.8)'; xb.font = 'bold 14px "JetBrains Mono"'; xb.textAlign = 'left'; const ly = h * 0.12 + (maxP - lastVis.c) / (maxP - minP) * h * 0.6; xb.fillText(`$${lastVis.c.toFixed(2)}`, w * 0.85, ly + 4); xb.textAlign = 'center'; }
			}
			if (phase === 'boom') {
				xa.globalCompositeOperation = 'lighter'; shards.forEach(s => { s.vy += 0.06; s.x += s.vx; s.y += s.vy; s.vx *= 0.997; s.rot += s.rv; s.life -= 0.004; if (s.life > 0) { xa.save(); xa.translate(s.x, s.y); xa.rotate(s.rot); xa.fillStyle = `rgba(${s.color.join(',')},${s.life * 0.9})`; xa.shadowBlur = 6; xa.shadowColor = `rgba(${s.color.join(',')},${s.life * 0.4})`; xa.fillRect(-s.size / 2, -s.size / 3, s.size, s.size * 0.6); xa.shadowBlur = 0; xa.restore(); } }); shards.filter(s => s.life > 0); xa.globalCompositeOperation = 'source-over';
				cloudParts.forEach(p => { p.vy -= 0.02; p.vx *= 0.995; p.x += p.vx; p.y += p.vy; p.life -= 0.003; p.size *= 1.002; if (p.life > 0) { xb.beginPath(); xb.arc(p.x, p.y, p.size, 0, Math.PI * 2); xb.fillStyle = `hsla(${p.hue},80%,50%,${p.life * 0.15})`; xb.fill(); } }); cloudParts.filter(p => p.life > 0);
				shockwaves.forEach(sw => { sw.r += 12; sw.alpha *= 0.97; xc.beginPath(); xc.arc(w / 2, h * 0.4, sw.r, 0, Math.PI * 2); xc.strokeStyle = `rgba(255,255,255,${sw.alpha})`; xc.lineWidth = sw.width; xc.stroke(); }); shockwaves.filter(sw => sw.alpha > 0.01);
			}
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		function detonate() {
			phase = 'boom';
			for (let i = 0; i < 3000; i++) { const a = Math.random() * Math.PI * 2, spd = 3 + Math.random() * 15, isGreen = Math.random() > 0.25; shards.push({ x: w * 0.17 + Math.random() * w * 0.65, y: h * 0.12 + Math.random() * h * 0.6, vx: Math.cos(a) * spd, vy: Math.sin(a) * spd, size: Math.random() * 6 + 2, rot: 0, rv: (Math.random() - 0.5) * 0.3, life: 1, color: isGreen ? [0, 255, 136] : [255, 220, 50] }); }
			for (let i = 0; i < 800; i++) { const a = Math.random() * Math.PI * 2, spd = Math.random() * 6 + 1; cloudParts.push({ x: w / 2, y: h * 0.4, vx: Math.cos(a) * spd, vy: Math.sin(a) * spd * 0.6 - Math.random() * 4, size: Math.random() * 20 + 8, life: 1, hue: 100 + Math.random() * 40 }); }
			shockwaves.push({ r: 0, alpha: 0.8, width: 6 }); setTimeout(() => shockwaves.push({ r: 0, alpha: 0.6, width: 4 }), 100);
		}
		const trigger = ScrollTrigger.create({ trigger: '#s10', start: 'top 60%', onEnter: () => { on = true; phase = 'build'; const tl = gsap.timeline(); candles.forEach((c, i) => { const delay = i < 30 ? i * 0.07 : 30 * 0.07 + (i - 30) * 0.04; tl.add(() => { c.visible = true; }, delay); }); const boomTime = 30 * 0.07 + 12 * 0.04 + 0.3; tl.add(() => detonate(), boomTime); tl.to('#s10f', { opacity: 1, duration: 0.04 }, boomTime); tl.to('#s10f', { opacity: 0, duration: 2, ease: 'power3.out' }, boomTime + 0.04); tl.add(() => shake(document.getElementById('s10-content')!, 60, 1), boomTime); tl.to('#s10t', { opacity: 1, duration: 0.6 }, boomTime + 0.5); tl.from('#s10t', { scale: 5, filter: 'blur(40px)', duration: 0.6, ease: 'power4.out' }, boomTime + 0.5); tl.to('#s10u', { opacity: 1, duration: 1.2 }, boomTime + 1.5); tl.to('#s10v', { opacity: 0.12, duration: 1 }, boomTime + 2); tl.to('#s10l', { opacity: 0.12, duration: 0.5 }, boomTime + 1); tl.to('#s10r', { opacity: 0.12, duration: 0.5 }, boomTime + 1.2); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initTheMove(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="s10">
	<canvas bind:this={s10a}></canvas>
	<canvas bind:this={s10b} class="c2"></canvas>
	<canvas bind:this={s10c} class="c3"></canvas>
	<canvas bind:this={s10d} class="c4"></canvas>
	<div class="flash" id="s10f"></div>
	<div class="bk" id="s10bk"></div>
	<div class="ct" id="s10-content">
		<h1 class="t1" id="s10t">THE MOVE PRIOR<br>TO THE MOVE</h1>
		<p class="t2" id="s10u">EXPLOSIVE SWINGS — REVOLUTION TRADING PROS</p>
		<p class="t3" id="s10v">[ 18,000+ TRADERS // EST. BY BILLY RIBEIRO ]</p>
	</div>
	<span class="lbl" id="s10l">10 — THE MOVE</span>
	<span class="rbl" id="s10r">REVOLUTION<br>TRADING<br>PROS</span>
</section>
<div style="height:10vh;background:#000"></div>

<style>
	:global(*) { margin: 0; padding: 0; box-sizing: border-box; }
	:global(html) { overflow-x: hidden; background: #000; }
	.hero { position: relative; width: 100%; height: 100vh; overflow: hidden; display: flex; align-items: center; justify-content: center; color: #fff; font-family: 'JetBrains Mono', monospace; }
	.hero canvas { position: absolute; inset: 0; width: 100%; height: 100%; z-index: 1; }
	.c2 { z-index: 2; } .c3 { z-index: 3; } .c4 { z-index: 4; }
	.flash { position: absolute; inset: 0; z-index: 30; pointer-events: none; opacity: 0; }
	.bk { position: absolute; inset: 0; z-index: 35; pointer-events: none; background: #000; opacity: 0; }
	.ct { position: relative; z-index: 20; text-align: center; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%; pointer-events: none; }
	.lbl { position: absolute; top: 25px; left: 30px; z-index: 60; font-size: 0.45rem; letter-spacing: 0.6em; text-transform: uppercase; opacity: 0.15; }
	.rbl { position: absolute; top: 25px; right: 30px; z-index: 60; font-size: 0.35rem; letter-spacing: 0.3em; text-align: right; opacity: 0; }
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(2.5rem, 8vw, 7rem); line-height: 0.82; letter-spacing: -0.01em; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.4rem, 1vw, 0.65rem); letter-spacing: 0.8em; margin-top: 15px; opacity: 0; }
	.t3 { font-family: 'JetBrains Mono', monospace; font-size: 0.45rem; letter-spacing: 0.4em; margin-top: 25px; opacity: 0; }
	#s10 { background: #000; } #s10f { background: #fff; }
	#s10t { background: linear-gradient(180deg, #fff, #00ff88, #006633); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#s10u { color: rgba(0,255,136,0.6); }
	#s10v { color: rgba(0,255,136,0.3); }
	#s10l { color: #00ff88; }
	#s10r { color: rgba(0,255,136,0.3); }
</style>
