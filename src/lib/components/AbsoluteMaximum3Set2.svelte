<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0, prevMx = 0, prevMy = 0;
	let s4a: HTMLCanvasElement, s4b: HTMLCanvasElement, s4c: HTMLCanvasElement, s4d: HTMLCanvasElement;
	let s5a: HTMLCanvasElement, s5b: HTMLCanvasElement, s5c: HTMLCanvasElement, s5d: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];
	let chartInterval: number;

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }
	function updateMouseVelocity() { prevMx = mx; prevMy = my; rafIds.push(requestAnimationFrame(updateMouseVelocity)); }
	function shake(el: HTMLElement, intensity = 20, dur = 0.5) { const s = Math.floor(dur / 0.016); for (let i = 0; i < s; i++) { const f = Math.pow(1 - i / s, 2); gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.6, delay: i * 0.016 }); } gsap.set(el, { x: 0, y: 0, delay: s * 0.016 }); }

	function initMove() {
		const cv = [s4a, s4b, s4c, s4d];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s4a.width = s4a.parentElement!.offsetWidth, h = s4a.height = s4a.parentElement!.offsetHeight;
		let t = 0, on = false, phase = 'idle';
		let candles: { o: number; c: number; h: number; l: number; bull: boolean; visible: boolean; breakout?: boolean }[] = [];
		let shards: { x: number; y: number; vx: number; vy: number; size: number; rot: number; rv: number; life: number; color: number[] }[] = [];
		let cloudParts: { x: number; y: number; vx: number; vy: number; size: number; life: number; hue: number }[] = [];
		let shockwaves: { r: number; alpha: number; width: number }[] = [];
		let price = 5850;
		function genChart() { candles = []; for (let i = 0; i < 30; i++) { const o = price; price += (Math.random() - 0.5) * 6; const c = price; candles.push({ o, c, h: Math.max(o, c) + Math.random() * 3, l: Math.min(o, c) - Math.random() * 3, bull: c > o, visible: false }); } for (let i = 0; i < 12; i++) { const o = price; price += Math.random() * 25 + 8; const c = price; candles.push({ o, c, h: c + Math.random() * 5, l: o - Math.random() * 2, bull: true, visible: false, breakout: true }); } }
		genChart();
		function detonate() {
			phase = 'boom';
			const prices = candles.flatMap(c => [c.h, c.l]), minP = Math.min(...prices) - 5, maxP = Math.max(...prices) + 5, gap = w * 0.65 / candles.length;
			for (let i = 0; i < 3000; i++) { const a = Math.random() * Math.PI * 2, spd = 3 + Math.random() * 15, isGreen = Math.random() > 0.25; shards.push({ x: w * 0.17 + Math.random() * w * 0.65, y: h * 0.12 + Math.random() * h * 0.6, vx: Math.cos(a) * spd, vy: Math.sin(a) * spd, size: Math.random() * 6 + 2, rot: 0, rv: (Math.random() - 0.5) * 0.3, life: 1, color: isGreen ? [0, 255, 136] : [255, 220, 50] }); }
			for (let i = 0; i < 800; i++) { const a = Math.random() * Math.PI * 2, spd = Math.random() * 6 + 1; cloudParts.push({ x: w / 2, y: h * 0.4, vx: Math.cos(a) * spd, vy: Math.sin(a) * spd * 0.6 - Math.random() * 4, size: Math.random() * 20 + 8, life: 1, hue: 100 + Math.random() * 40 }); }
			shockwaves.push({ r: 0, alpha: 0.8, width: 6 }); timeouts.push(window.setTimeout(() => shockwaves.push({ r: 0, alpha: 0.6, width: 4 }), 100));
		}
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s4a.width; h = s4a.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; const [xa, xb, xc, xd] = ctx;
			xa.fillStyle = `rgba(0,0,0,${phase === 'boom' ? 0.06 : 0.1})`; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xc.clearRect(0, 0, w, h); xd.clearRect(0, 0, w, h);
			if (phase === 'build') {
				const vis = candles.filter(c => c.visible); if (vis.length === 0) { rafIds.push(requestAnimationFrame(draw)); return; }
				const prices = vis.flatMap(c => [c.h, c.l]), minP = Math.min(...prices) - 5, maxP = Math.max(...prices) + 5, range = maxP - minP, gap = w * 0.65 / candles.length, cw = Math.min(gap * 0.65, 11), offX = w * 0.17;
				vis.forEach((c, idx) => { const i = candles.indexOf(c), x = offX + i * gap + gap / 2, oY = h * 0.12 + (maxP - c.o) / range * h * 0.6, cY = h * 0.12 + (maxP - c.c) / range * h * 0.6, hY = h * 0.12 + (maxP - c.h) / range * h * 0.6, lY = h * 0.12 + (maxP - c.l) / range * h * 0.6, green = c.bull, color = green ? '0,255,136' : '255,51,68'; xb.strokeStyle = `rgba(${color},0.6)`; xb.lineWidth = 1.5; xb.beginPath(); xb.moveTo(x, hY); xb.lineTo(x, lY); xb.stroke(); const top = Math.min(oY, cY), bh = Math.abs(cY - oY) || 1.5; xb.fillStyle = `rgba(${color},0.85)`; xb.fillRect(x - cw / 2, top, cw, bh); if (c.breakout) { xb.shadowBlur = 25; xb.shadowColor = 'rgba(0,255,136,0.7)'; xb.fillRect(x - cw / 2, top, cw, bh); xb.shadowBlur = 0; } });
				const lastVis = vis[vis.length - 1]; if (lastVis) { xb.fillStyle = 'rgba(0,255,136,0.8)'; xb.font = 'bold 14px "JetBrains Mono"'; const ly = h * 0.12 + (maxP - lastVis.c) / (maxP - minP) * h * 0.6; xb.fillText(`$${lastVis.c.toFixed(2)}`, w * 0.85, ly + 4); }
			}
			if (phase === 'boom') {
				xa.globalCompositeOperation = 'lighter';
				shards.forEach(s => { s.vy += 0.06; s.x += s.vx; s.y += s.vy; s.vx *= 0.997; s.rot += s.rv; s.life -= 0.004; if (s.life > 0) { xa.save(); xa.translate(s.x, s.y); xa.rotate(s.rot); xa.fillStyle = `rgba(${s.color.join(',')},${s.life * 0.9})`; xa.shadowBlur = 6; xa.shadowColor = `rgba(${s.color.join(',')},${s.life * 0.4})`; xa.fillRect(-s.size / 2, -s.size / 3, s.size, s.size * 0.6); xa.shadowBlur = 0; xa.restore(); } }); shards = shards.filter(s => s.life > 0);
				cloudParts.forEach(p => { p.vy -= 0.02; p.vx *= 0.995; p.x += p.vx; p.y += p.vy; p.life -= 0.003; p.size *= 1.002; if (p.life > 0) { xa.beginPath(); xa.arc(p.x, p.y, p.size, 0, Math.PI * 2); xa.fillStyle = `hsla(${p.hue},80%,50%,${p.life * 0.15})`; xa.fill(); } }); cloudParts = cloudParts.filter(p => p.life > 0); xa.globalCompositeOperation = 'source-over';
				shockwaves.forEach(sw => { sw.r += 12; sw.alpha *= 0.97; xc.beginPath(); xc.arc(w / 2, h * 0.4, sw.r, 0, Math.PI * 2); xc.strokeStyle = `rgba(255,255,255,${sw.alpha})`; xc.lineWidth = sw.width; xc.stroke(); }); shockwaves = shockwaves.filter(sw => sw.alpha > 0.01);
			}
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s4', start: 'top 60%', onEnter: () => { on = true; phase = 'build'; const tl = gsap.timeline(); candles.forEach((c, i) => { const delay = i < 30 ? i * 0.07 : 30 * 0.07 + (i - 30) * 0.04; tl.add(() => { c.visible = true; }, delay); }); const boomTime = 30 * 0.07 + 12 * 0.04 + 0.3; tl.add(() => detonate(), boomTime); tl.to('#s4f', { opacity: 1, duration: 0.04 }, boomTime); tl.to('#s4f', { opacity: 0, duration: 2, ease: 'power3.out' }, boomTime + 0.04); tl.add(() => shake(document.getElementById('s4-content')!, 60, 1), boomTime); tl.to('#s4t', { opacity: 1, duration: 0.6 }, boomTime + 0.5); tl.from('#s4t', { scale: 5, filter: 'blur(40px)', duration: 0.6, ease: 'power4.out' }, boomTime + 0.5); tl.to('#s4s', { opacity: 1, duration: 1.2 }, boomTime + 1.5); } }); triggers.push(trigger);
	}

	function initMatrix() {
		const cv = [s5a, s5b, s5c, s5d];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s5a.width = s5a.parentElement!.offsetWidth, h = s5a.height = s5a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const CHARS = 'アイウエオカキクケコサシスセソタチツテトナニヌネノ0123456789ABCDEF$€¥∞Γ'; const charArr = CHARS.split('');
		const fontSize = 16; let cols = Math.ceil(w / fontSize), drops: number[] = [], colSpeed: number[] = [], colBright: number[] = [];
		function initMatrixData() { cols = Math.ceil((w || 1920) / fontSize); drops = []; colSpeed = []; colBright = []; for (let i = 0; i < cols; i++) { drops.push(Math.random() * -100); colSpeed.push(Math.random() * 0.5 + 0.5); colBright.push(Math.random()); } }
		initMatrixData();
		const candles: { x: number; o: number; c: number; h: number; l: number; visible: boolean }[] = [];
		for (let i = 0; i < 20; i++) candles.push({ x: (i / 20) * (w * 0.7) + w * 0.15, o: h * 0.6, c: h * 0.4, h: h * 0.35, l: h * 0.65, visible: false });
		function addCandle() { const idx = candles.findIndex(c => !c.visible); if (idx >= 0) { const c = candles[idx], prev = candles[idx - 1]; let base = prev ? prev.c : h * 0.5; const change = (Math.random() - 0.5) * h * 0.15; c.o = base; c.c = base + change; c.h = Math.min(c.o, c.c) - Math.random() * h * 0.05; c.l = Math.max(c.o, c.c) + Math.random() * h * 0.05; c.visible = true; } }
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s5a.width; h = s5a.height; initMatrixData(); }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.01; const [xa, xb, xc, xd] = ctx;
			xa.fillStyle = 'rgba(0,0,0,0.08)'; xa.fillRect(0, 0, w, h);
			xa.font = `${fontSize}px "JetBrains Mono"`; xa.textAlign = 'center';
			for (let i = 0; i < cols; i++) {
				const x = i * fontSize, speed = colSpeed[i], bright = colBright[i];
				colBright[i] = bright + Math.sin(t * 3 + i) * 0.02;
				for (let j = 0; j < 3; j++) { const y = (drops[i] - j * fontSize * 0.7) % h; if (y > 0) { const head = j === 0, a = (head ? 1 : 0.3) * (0.5 + colBright[i] * 0.5), char = charArr[Math.floor(Math.random() * charArr.length)]; xa.fillStyle = head ? `rgba(255,255,255,${a})` : `rgba(0,255,136,${a})`; if (head) { xa.shadowBlur = 10; xa.shadowColor = `rgba(0,255,136,${a * 0.5})`; } xa.fillText(char, x, y); xa.shadowBlur = 0; } }
				drops[i] += speed;
			}
			xb.clearRect(0, 0, w, h);
			candles.forEach(c => { if (!c.visible) return; const green = c.c < c.o, color = green ? '0,255,136' : '255,51,68', cw = (w * 0.7 / 20) * 0.6; xb.strokeStyle = `rgba(${color},0.8)`; xb.lineWidth = 1; xb.beginPath(); xb.moveTo(c.x, c.h); xb.lineTo(c.x, c.l); xb.stroke(); xb.fillStyle = `rgba(${color},0.9)`; const top = Math.min(c.o, c.c), ch = Math.abs(c.c - c.o) || 1; xb.fillRect(c.x - cw / 2, top, cw, ch); });
			xb.strokeStyle = 'rgba(0,255,136,0.15)'; xb.setLineDash([4, 8]); xb.lineWidth = 0.5; [0.25, 0.5, 0.75].forEach(lvl => { const ly = h * 0.2 + lvl * h * 0.6; xb.beginPath(); xb.moveTo(w * 0.1, ly); xb.lineTo(w * 0.9, ly); xb.stroke(); }); xb.setLineDash([]);
			const lastC = candles.filter(c => c.visible).pop(); if (lastC) { xb.fillStyle = 'rgba(0,255,136,0.9)'; xb.font = 'bold 12px "JetBrains Mono"'; xb.textAlign = 'left'; const pct = ((lastC.o - lastC.c) / lastC.o * 100).toFixed(2); xb.fillText(`${pct}%`, lastC.x + 15, lastC.c); xb.textAlign = 'center'; }
			xc.clearRect(0, 0, w, h); xc.fillStyle = 'rgba(0,0,0,0.02)'; xc.fillRect(w * 0.1, h * 0.1, w * 0.8, h * 0.05);
			const greenPct = candles.filter(c => c.visible && c.c < c.o).length / Math.max(1, candles.filter(c => c.visible).length);
			const grd = xc.createLinearGradient(w * 0.1, 0, w * 0.9, 0); grd.addColorStop(0, 'rgba(255,0,0,0.6)'); grd.addColorStop(greenPct, 'rgba(0,255,136,0.6)'); grd.addColorStop(1, 'rgba(0,255,0,0.6)');
			xc.fillStyle = grd; xc.fillRect(w * 0.1, h * 0.1, w * 0.8 * (candles.filter(c => c.visible).length / candles.length), h * 0.05);
			xc.fillStyle = 'rgba(0,255,136,0.8)'; xc.font = '9px "JetBrains Mono"'; xc.textAlign = 'right'; xc.fillText(`RTP: ${(50 + greenPct * 50).toFixed(1)}%`, w * 0.9, h * 0.135); xc.textAlign = 'center';
			xd.clearRect(0, 0, w, h);
			['GAMMA', 'THETA', 'VEGA', 'DELTA', 'RHO'].forEach((g, i) => { const gx = w * 0.15 + i * w * 0.17, gy = h - 60 + Math.sin(t * 2 + i) * 10; const glow = 0.3 + Math.sin(t * 3 + i) * 0.2; xd.fillStyle = `rgba(0,200,255,${glow})`; xd.font = 'bold 9px "JetBrains Mono"'; xd.fillText(g, gx, gy); });
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s5', start: 'top 60%', onEnter: () => { on = true; const tl = gsap.timeline(); chartInterval = window.setInterval(addCandle, 100); tl.add(() => { addCandle(); addCandle(); }, 0.5); tl.to('#s5t', { opacity: 1, duration: 0.6 }, 1); tl.from('#s5t', { scale: 2, filter: 'blur(30px)', duration: 0.6, ease: 'power4.out' }, 1); tl.add(() => shake(document.getElementById('s5-content')!, 25, 0.5), 1); tl.to('#s5s', { opacity: 1, duration: 1 }, 1.8); tl.to('#s5s2', { opacity: 1, duration: 1 }, 2.5); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; prevMx = mx; prevMy = my; rafIds.push(requestAnimationFrame(updateMouseVelocity)); initMove(); initMatrix(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); if (chartInterval) clearInterval(chartInterval); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="s4" style="background:#000a05">
	<canvas bind:this={s4a}></canvas>
	<canvas bind:this={s4b} class="c2"></canvas>
	<canvas bind:this={s4c} class="c3"></canvas>
	<canvas bind:this={s4d} class="c4"></canvas>
	<div class="flash" id="s4f" style="background:#fff"></div>
	<div class="ct" id="s4-content">
		<h1 class="t1" id="s4t">THE MOVE</h1>
		<p class="t2" id="s4s">MARKET NUKE INCOMING.</p>
	</div>
	<span class="lbl">04 — BREAKOUT</span>
	<span class="idx">04</span>
</section>
<div class="sp"></div>

<section class="hero" id="s5" style="background:#000208">
	<canvas bind:this={s5a}></canvas>
	<canvas bind:this={s5b} class="c2"></canvas>
	<canvas bind:this={s5c} class="c3"></canvas>
	<canvas bind:this={s5d} class="c4"></canvas>
	<div class="ct" id="s5-content">
		<h1 class="t1" id="s5t">MATRIX OMEGA</h1>
		<p class="t2" id="s5s">THE SYSTEM IS RIGGED. PLAY ANYWAY.</p>
		<p class="t3" id="s5s2">[ RTP: CALCULATING... ]</p>
	</div>
	<span class="lbl">05 — GRAND FINALE</span>
	<span class="idx">05</span>
</section>

<style>
	:global(*) { margin: 0; padding: 0; box-sizing: border-box; }
	:global(html) { overflow-x: hidden; background: #000; }
	.hero { position: relative; width: 100%; height: 100vh; overflow: hidden; display: flex; align-items: center; justify-content: center; color: #fff; font-family: 'JetBrains Mono', monospace; }
	.hero canvas { position: absolute; inset: 0; width: 100%; height: 100%; z-index: 1; }
	.c2 { z-index: 2; } .c3 { z-index: 3; } .c4 { z-index: 4; }
	.flash { position: absolute; inset: 0; z-index: 30; pointer-events: none; opacity: 0; }
	.ct { position: relative; z-index: 10; text-align: center; pointer-events: none; }
	.lbl { position: absolute; top: 25px; left: 30px; z-index: 60; font-size: 0.45rem; letter-spacing: 0.6em; text-transform: uppercase; opacity: 0.15; }
	.idx { position: absolute; bottom: 25px; right: 35px; z-index: 60; font-family: 'Bebas Neue'; font-size: 5rem; line-height: 1; opacity: 0.03; }
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(3.5rem, 13vw, 11rem); line-height: 0.85; letter-spacing: 0.02em; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.45rem, 1.1vw, 0.7rem); letter-spacing: 0.7em; margin-top: 18px; opacity: 0; }
	.t3 { font-family: 'JetBrains Mono', monospace; font-size: 0.5rem; letter-spacing: 0.35em; margin-top: 28px; opacity: 0; }
	.sp { height: 2px; background: #000; }
	#s4 { background: #000a05; } #s5 { background: #000208; }
</style>
