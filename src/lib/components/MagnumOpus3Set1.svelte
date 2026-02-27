<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let s1a: HTMLCanvasElement, s1b: HTMLCanvasElement, s1c: HTMLCanvasElement;
	let s2a: HTMLCanvasElement, s2b: HTMLCanvasElement, s2c: HTMLCanvasElement;
	let s3a: HTMLCanvasElement, s3b: HTMLCanvasElement, s3c: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }
	function shake(el: HTMLElement, intensity = 20, dur = 0.5) { const s = Math.floor(dur / 0.016); for (let i = 0; i < s; i++) { const f = Math.pow(1 - i / s, 2); gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.6, delay: i * 0.016 }); } gsap.set(el, { x: 0, y: 0, delay: s * 0.016 }); }

	function initGenesis() {
		const cv = [s1a, s1b, s1c];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s1a.width = s1a.parentElement!.offsetWidth, h = s1a.height = s1a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const bang = { v: 0 };
		const stars: { a: number; d: number; speed: number; fx: number; fy: number; x: number; y: number; size: number; hue: number; bright: number }[] = [];
		for (let i = 0; i < 3000; i++) { const a = Math.random() * Math.PI * 2, d = Math.random(); stars.push({ a, d, speed: 2 + Math.random() * 10, fx: Math.cos(a) * (50 + d * Math.min(w || 1920, 1080) * 0.6), fy: Math.sin(a) * (50 + d * Math.min(w || 1920, 1080) * 0.6) * 0.5, x: 0, y: 0, size: Math.random() * 2.5 + 0.5, hue: 30 + Math.random() * 40, bright: Math.random() * 0.5 + 0.5 }); }
		const spiral: { angle: number; dist: number; speed: number; size: number; hue: number }[] = [];
		for (let i = 0; i < 500; i++) { const arm = Math.floor(Math.random() * 3), dist = Math.random() * 300 + 50, angle = arm * (Math.PI * 2 / 3) + dist * 0.008 + Math.random() * 0.3; spiral.push({ angle, dist, speed: 0.001 + Math.random() * 0.002, size: Math.random() * 2 + 1, hue: 20 + Math.random() * 50 }); }
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s1a.width; h = s1a.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; const B = bang.v; const [xa, xb, xc] = ctx; const CX = w / 2, CY = h / 2;
			xa.fillStyle = `rgba(0,0,0,${B < 1 ? 0.3 : 0.08})`; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xc.clearRect(0, 0, w, h);
			if (B <= 0) { rafIds.push(requestAnimationFrame(draw)); return; }
			xa.globalCompositeOperation = 'lighter'; const mx_pos = (mx / w - 0.5) * 40, my_pos = (my / h - 0.5) * 40;
			stars.forEach(s => { const progress = Math.min(1, B); s.x = s.fx * progress; s.y = s.fy * progress; const px = CX + s.x + mx_pos * s.d, py = CY + s.y + my_pos * s.d; if (px < -20 || px > w + 20 || py < -20 || py > h + 20) return; const alpha = s.bright * Math.min(1, B) * (0.5 + Math.sin(t * 5 + s.a) * 0.3); xa.beginPath(); xa.arc(px, py, s.size * (B < 0.3 ? B / 0.3 : 1), 0, Math.PI * 2); xa.fillStyle = `hsla(${s.hue},80%,${60 + s.bright * 30}%,${alpha})`; xa.fill(); if (B < 2 && B > 0.1) { const streak = Math.min(B, 1) * 15 * s.speed / 10; const angle = Math.atan2(s.y, s.x); xa.beginPath(); xa.moveTo(px, py); xa.lineTo(px - Math.cos(angle) * streak, py - Math.sin(angle) * streak); xa.strokeStyle = `hsla(${s.hue},80%,70%,${alpha * 0.6})`; xa.lineWidth = s.size * 0.6; xa.stroke(); } });
			xa.globalCompositeOperation = 'source-over';
			if (B > 2) { xb.globalCompositeOperation = 'lighter'; const sF = Math.min(1, (B - 2) / 3); spiral.forEach(s => { s.angle += s.speed; const px = CX + Math.cos(s.angle) * s.dist * sF + mx_pos * 0.5, py = CY + Math.sin(s.angle) * s.dist * 0.45 * sF + my_pos * 0.5; xb.beginPath(); xb.arc(px, py, s.size * sF, 0, Math.PI * 2); xb.fillStyle = `hsla(${s.hue},80%,60%,${sF * 0.5})`; xb.shadowBlur = 6; xb.shadowColor = `hsla(${s.hue},100%,50%,0.3)`; xb.fill(); xb.shadowBlur = 0; }); xb.globalCompositeOperation = 'source-over'; }
			const coreR = B < 1 ? B * 100 : 100 - Math.min(80, (B - 1) * 20); const grd = xc.createRadialGradient(CX, CY, 0, CX, CY, coreR); grd.addColorStop(0, `rgba(255,220,150,${Math.min(0.8, B * 0.4)})`); grd.addColorStop(0.3, `rgba(255,150,50,${Math.min(0.3, B * 0.15)})`); grd.addColorStop(1, 'transparent'); xc.fillStyle = grd; xc.fillRect(0, 0, w, h);
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s1', start: 'top 60%', onEnter: () => { on = true; const tl = gsap.timeline(); tl.set('#s1bk', { opacity: 1 }); tl.to('#s1bk', { opacity: 0.95, duration: 1 }, 1); tl.to(bang, { v: 5, duration: 4, ease: 'power2.out' }, 2); tl.to('#s1bk', { opacity: 0, duration: 0.05 }, 2); tl.set('#s1f', { opacity: 1 }, 2); tl.to('#s1f', { opacity: 0, duration: 2.5, ease: 'power3.out' }, 2.05); tl.add(() => shake(document.getElementById('s1-content')!, 50, 0.8), 2); tl.to('#s1t', { opacity: 1, duration: 0.5 }, 4); tl.from('#s1t', { scale: 6, filter: 'blur(30px)', duration: 0.6, ease: 'power4.out' }, 4); tl.to('#s1u', { opacity: 1, duration: 1 }, 4.8); tl.to('#s1l', { opacity: 0.12, duration: 0.5 }, 4.5); } }); triggers.push(trigger);
	}

	function initTsunami() {
		const cv = [s2a, s2b, s2c];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s2a.width = s2a.parentElement!.offsetWidth, h = s2a.height = s2a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const waveX = { v: -200 };
		const foam: { x: number; y: number; vx: number; vy: number; size: number; life: number; hue: number }[] = [];
		for (let i = 0; i < 800; i++) foam.push({ x: -100, y: 0, vx: 0, vy: 0, size: Math.random() * 5 + 1, life: 0, hue: 140 + Math.random() * 30 });
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s2a.width; h = s2a.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; const [xa, xb, xc] = ctx; const WX = waveX.v;
			xa.fillStyle = 'rgba(0,3,8,0.08)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xc.clearRect(0, 0, w, h);
			for (let i = 0; i < 8; i++) { const cx2 = w * 0.1 + i * w * 0.12 + Math.sin(t + i * 0.7) * 40, cy2 = h * 0.2 + Math.sin(t * 0.5 + i) * 30; const grd = xa.createRadialGradient(cx2, cy2, 0, cx2, cy2, 80); grd.addColorStop(0, 'rgba(0,100,130,0.03)'); grd.addColorStop(1, 'transparent'); xa.fillStyle = grd; xa.fillRect(cx2 - 80, cy2 - 80, 160, 160); }
			xb.globalCompositeOperation = 'lighter'; const waveH = h * 0.7, numCandles = 60;
			for (let i = 0; i < numCandles; i++) { const candleX = WX - i * 15; if (candleX < -30 || candleX > w + 100) continue; const wavePhase = i / numCandles, candleH = waveH * Math.pow(1 - wavePhase, 1.5) * (0.3 + Math.sin(t * 3 + i * 0.2) * 0.15), candleY = h * 0.85 - candleH, candleW = 12; const bodyAlpha = 0.15 + wavePhase * 0.1; xb.fillStyle = `rgba(0,255,136,${bodyAlpha})`; xb.fillRect(candleX - candleW / 2, candleY, candleW, candleH); if (i < 15) { xb.shadowBlur = 20; xb.shadowColor = `rgba(0,255,136,${(1 - i / 15) * 0.4})`; xb.fillRect(candleX - candleW / 2, candleY, candleW, candleH); xb.shadowBlur = 0; } xb.fillStyle = `rgba(0,255,180,${bodyAlpha * 0.7})`; xb.fillRect(candleX - 1, candleY - Math.random() * 20 - 5, 2, Math.random() * 20 + 5); }
			const crestX = WX, crestY = h * 0.15 + Math.sin(t * 4) * 20; xb.beginPath(); for (let i = 0; i < 40; i++) { const fx = crestX - i * 5 + (Math.random() - 0.5) * 10, fy = crestY + Math.pow(i / 40, 2) * h * 0.7 + (Math.random() - 0.5) * 15; xb.moveTo(fx, fy); xb.arc(fx, fy, Math.random() * 6 + 2, 0, Math.PI * 2); } xb.fillStyle = 'rgba(150,255,200,0.3)'; xb.fill(); xb.globalCompositeOperation = 'source-over';
			foam.forEach(f => { if (f.life <= 0 && Math.abs(crestX - w / 2) < w) { f.x = crestX + (Math.random() - 0.5) * 40; f.y = crestY + Math.random() * 40; f.vx = Math.random() * 6 + 2; f.vy = -Math.random() * 8 - 3; f.life = 1; } if (f.life > 0) { f.vy += 0.15; f.x += f.vx; f.y += f.vy; f.life -= 0.012; xc.beginPath(); xc.arc(f.x, f.y, f.size * f.life, 0, Math.PI * 2); xc.fillStyle = `hsla(${f.hue},100%,70%,${f.life * 0.6})`; xc.shadowBlur = f.size * 3; xc.shadowColor = `hsla(${f.hue},100%,50%,${f.life * 0.3})`; xc.fill(); xc.shadowBlur = 0; } });
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s2', start: 'top 60%', onEnter: () => { on = true; const tl = gsap.timeline(); tl.to(waveX, { v: w + 300, duration: 4, ease: 'power2.in' }, 0); tl.to('#s2f', { opacity: 0.6, duration: 0.1 }, 0.5); tl.to('#s2f', { opacity: 0, duration: 1.5 }, 0.6); tl.to('#s2t', { opacity: 1, duration: 0.5 }, 2); tl.from('#s2t', { x: -200, filter: 'blur(20px)', duration: 0.5, ease: 'power4.out' }, 2); tl.add(() => shake(document.getElementById('s2-content')!, 20, 0.4), 2); tl.to('#s2u', { opacity: 1, duration: 1 }, 2.8); tl.to('#s2l', { opacity: 0.12, duration: 0.5 }, 2.5); } }); triggers.push(trigger);
	}

	function initEclipse() {
		const cv = [s3a, s3b, s3c];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s3a.width = s3a.parentElement!.offsetWidth, h = s3a.height = s3a.parentElement!.offsetHeight;
		let t = 0, on = false, totality = 0;
		const moonX = { v: 0 };
		const stars: { x: number; y: number; s: number; a: number; tw: number }[] = []; for (let i = 0; i < 600; i++) stars.push({ x: Math.random() * 2000, y: Math.random() * 1200, s: Math.random() * 2 + 0.3, a: Math.random() * 0.7 + 0.1, tw: Math.random() * 10 });
		const corona: { a: number; d: number; size: number; bright: number; speed: number }[] = []; for (let i = 0; i < 1500; i++) { const a = Math.random() * Math.PI * 2, d = 80 + Math.random() * 200; corona.push({ a, d, size: Math.random() * 2 + 0.5, bright: Math.random() * 0.5 + 0.5, speed: (Math.random() - 0.5) * 0.002 }); }
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s3a.width; h = s3a.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.006; const [xa, xb, xc] = ctx; const CX = w / 2 + (mx / w - 0.5) * 20, CY = h * 0.4 + (my / h - 0.5) * 15; const sunR = 70;
			xa.fillStyle = 'rgba(0,0,3,0.1)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xc.clearRect(0, 0, w, h);
			const starAlpha = totality; stars.forEach(s => { const tw = 0.5 + Math.sin(t * 3 + s.tw) * 0.5; xa.beginPath(); xa.arc(s.x / 2000 * w, s.y / 1200 * h, s.s, 0, Math.PI * 2); xa.fillStyle = `rgba(255,255,255,${s.a * tw * starAlpha})`; xa.fill(); });
			const sunGlow = xb.createRadialGradient(CX, CY, sunR * 0.8, CX, CY, sunR * 4); sunGlow.addColorStop(0, `rgba(255,200,100,${0.3 * (1 - totality * 0.5)})`); sunGlow.addColorStop(0.3, `rgba(255,150,50,${0.1 * (1 - totality * 0.3)})`); sunGlow.addColorStop(1, 'transparent'); xb.fillStyle = sunGlow; xb.fillRect(0, 0, w, h);
			xb.beginPath(); xb.arc(CX, CY, sunR, 0, Math.PI * 2); xb.fillStyle = `rgba(255,220,150,${0.9 - totality * 0.7})`; xb.shadowBlur = 60; xb.shadowColor = `rgba(255,180,80,${0.6 - totality * 0.4})`; xb.fill(); xb.shadowBlur = 0;
			if (totality > 0.3) { xb.globalCompositeOperation = 'lighter'; corona.forEach(c => { c.a += c.speed; const d = c.d + Math.sin(t + c.a * 3) * 15, px = CX + Math.cos(c.a) * d, py = CY + Math.sin(c.a) * d; const coronaAlpha = c.bright * (totality - 0.3) / 0.7 * 0.6; const angle = Math.atan2(py - CY, px - CX); xb.beginPath(); xb.moveTo(px - Math.cos(angle) * c.size * 3, py - Math.sin(angle) * c.size * 3); xb.lineTo(px + Math.cos(angle) * c.size * 3, py + Math.sin(angle) * c.size * 3); xb.strokeStyle = `rgba(255,200,150,${coronaAlpha})`; xb.lineWidth = c.size; xb.stroke(); xb.beginPath(); xb.arc(px, py, c.size, 0, Math.PI * 2); xb.fillStyle = `rgba(255,220,180,${coronaAlpha * 0.8})`; xb.fill(); }); xb.globalCompositeOperation = 'source-over'; }
			const moonOff = moonX.v, moonCX = CX + moonOff; xc.beginPath(); xc.arc(moonCX, CY, sunR + 2, 0, Math.PI * 2); xc.fillStyle = '#000'; xc.fill(); xc.beginPath(); xc.arc(moonCX + 3, CY, sunR + 1, 0, Math.PI * 2); xc.strokeStyle = `rgba(200,200,220,${totality * 0.1})`; xc.lineWidth = 1; xc.stroke();
			if (totality > 0.5 && totality < 1) { const diamondSide = moonOff > 0 ? -1 : 1; const dx = CX + diamondSide * (sunR - 3), dy = CY; xc.beginPath(); xc.arc(dx, dy, 4 + Math.sin(t * 10) * 2, 0, Math.PI * 2); xc.fillStyle = `rgba(255,255,255,${(1 - Math.abs(totality - 0.75) * 4) * 0.9})`; xc.shadowBlur = 40; xc.shadowColor = 'rgba(255,220,150,0.8)'; xc.fill(); xc.shadowBlur = 0; }
			totality = Math.max(0, 1 - Math.abs(moonOff) / (sunR * 1.2));
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s3', start: 'top 60%', onEnter: () => { on = true; const tl = gsap.timeline(); tl.fromTo(moonX, { v: sunR * 3 }, { v: -sunR * 3, duration: 8, ease: 'power1.inOut' }, 0); tl.to('#s3f', { opacity: 0.6, duration: 0.08 }, 3.5); tl.to('#s3f', { opacity: 0, duration: 1.5 }, 3.6); tl.to('#s3t', { opacity: 1, duration: 0.8 }, 3.8); tl.from('#s3t', { y: -40, filter: 'blur(10px)', duration: 0.8, ease: 'power3.out' }, 3.8); tl.to('#s3u', { opacity: 1, duration: 1 }, 4.5); tl.to('#s3l', { opacity: 0.12, duration: 0.5 }, 4); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initGenesis(); initTsunami(); initEclipse(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="s1">
	<canvas bind:this={s1a}></canvas>
	<canvas bind:this={s1b} class="c2"></canvas>
	<canvas bind:this={s1c} class="c3"></canvas>
	<div class="flash" id="s1f"></div>
	<div class="bk" id="s1bk"></div>
	<div class="ct" id="s1-content">
		<h1 class="t1" id="s1t">GENESIS</h1>
		<p class="t2" id="s1u">IN THE BEGINNING, THERE WAS THE CHART</p>
	</div>
	<span class="lbl" id="s1l">01 — CREATION</span>
</section>
<div class="sp"></div>

<section class="hero" id="s2">
	<canvas bind:this={s2a}></canvas>
	<canvas bind:this={s2b} class="c2"></canvas>
	<canvas bind:this={s2c} class="c3"></canvas>
	<div class="flash" id="s2f"></div>
	<div class="ct" id="s2-content">
		<h1 class="t1" id="s2t">TSUNAMI</h1>
		<p class="t2" id="s2u">THE WAVE THAT DROWNS THE SHORTS</p>
	</div>
	<span class="lbl" id="s2l">02 — GREEN WAVE</span>
</section>
<div class="sp"></div>

<section class="hero" id="s3">
	<canvas bind:this={s3a}></canvas>
	<canvas bind:this={s3b} class="c2"></canvas>
	<canvas bind:this={s3c} class="c3"></canvas>
	<div class="flash" id="s3f"></div>
	<div class="ct" id="s3-content">
		<h1 class="t1" id="s3t">ECLIPSE</h1>
		<p class="t2" id="s3u">DARKNESS BEFORE THE BREAKOUT</p>
	</div>
	<span class="lbl" id="s3l">03 — ALIGNMENT</span>
</section>

<style>
	:global(*) { margin: 0; padding: 0; box-sizing: border-box; }
	:global(html) { overflow-x: hidden; background: #000; }
	.hero { position: relative; width: 100%; height: 100vh; overflow: hidden; display: flex; align-items: center; justify-content: center; color: #fff; font-family: 'JetBrains Mono', monospace; }
	.hero canvas { position: absolute; inset: 0; width: 100%; height: 100%; z-index: 1; }
	.c2 { z-index: 2; } .c3 { z-index: 3; }
	.flash { position: absolute; inset: 0; z-index: 30; pointer-events: none; opacity: 0; }
	.bk { position: absolute; inset: 0; z-index: 35; pointer-events: none; background: #000; opacity: 0; }
	.ct { position: relative; z-index: 20; text-align: center; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%; pointer-events: none; }
	.lbl { position: absolute; top: 25px; left: 30px; z-index: 60; font-size: 0.45rem; letter-spacing: 0.6em; text-transform: uppercase; opacity: 0.15; }
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(4rem, 15vw, 14rem); line-height: 0.82; letter-spacing: -0.01em; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.4rem, 1vw, 0.65rem); letter-spacing: 0.8em; margin-top: 15px; opacity: 0; }
	.sp { height: 2px; background: #000; }
	#s1 { background: #000; } #s1f { background: radial-gradient(circle, #fff, rgba(255,200,100,0.6) 30%, transparent 60%); }
	#s1t { background: linear-gradient(180deg, #fff, #ffd700, #ff6600); -webkit-background-clip: text; background-clip: text; color: transparent; }
	#s1u { color: rgba(255,200,100,0.6); } #s1l { color: #ffd700; }
	#s2 { background: #000308; } #s2f { background: linear-gradient(90deg, transparent, rgba(0,255,136,0.4), transparent); }
	#s2t { color: #00ff88; text-shadow: 0 0 80px rgba(0,255,136,0.5), 0 0 200px rgba(0,200,100,0.2); }
	#s2u { color: rgba(0,255,136,0.5); } #s2l { color: #00ff88; }
	#s3 { background: #000208; } #s3f { background: radial-gradient(circle at 50% 40%, rgba(255,200,100,0.8), transparent 30%); }
	#s3t { color: #fff; text-shadow: 0 0 60px rgba(255,200,100,0.6), 0 0 150px rgba(255,150,50,0.3); }
	#s3u { color: rgba(255,200,100,0.5); } #s3l { color: #ffd700; }
</style>
