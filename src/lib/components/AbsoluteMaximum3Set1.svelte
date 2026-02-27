<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0, prevMx = 0, prevMy = 0;
	let s1a: HTMLCanvasElement, s1b: HTMLCanvasElement, s1c: HTMLCanvasElement, s1d: HTMLCanvasElement;
	let s2a: HTMLCanvasElement, s2b: HTMLCanvasElement, s2c: HTMLCanvasElement;
	let s3a: HTMLCanvasElement, s3b: HTMLCanvasElement, s3c: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }
	function updateMouseVelocity() { prevMx = mx; prevMy = my; rafIds.push(requestAnimationFrame(updateMouseVelocity)); }
	function shake(el: HTMLElement, intensity = 20, dur = 0.5) { const s = Math.floor(dur / 0.016); for (let i = 0; i < s; i++) { const f = Math.pow(1 - i / s, 2); gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.6, delay: i * 0.016 }); } gsap.set(el, { x: 0, y: 0, delay: s * 0.016 }); }

	function initSingularity() {
		const cv = [s1a, s1b, s1c, s1d];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s1a.width = s1a.parentElement!.offsetWidth, h = s1a.height = s1a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const warp = { v: 0 };
		const stars: { x: number; y: number; z: number; size: number; hue: number }[] = [];
		for (let i = 0; i < 4000; i++) stars.push({ x: (Math.random() - 0.5) * 3, y: (Math.random() - 0.5) * 3, z: Math.random() * 5 + 0.5, size: Math.random() * 1.8 + 0.5, hue: 200 + Math.random() * 60 });
		const disk: { dist: number; angle: number; speed: number; tilt: number; size: number; hue: number; bright: number }[] = [];
		for (let i = 0; i < 2000; i++) { const d = 40 + Math.pow(Math.random(), 0.5) * 280; disk.push({ dist: d, angle: Math.random() * Math.PI * 2, speed: (1.2 / Math.sqrt(d / 60)) * (0.7 + Math.random() * 0.6), tilt: 0.28 + Math.random() * 0.08, size: Math.random() * 3 + 1, hue: 15 + Math.random() * 35, bright: 0.5 + Math.random() * 0.5 }); }
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s1a.width; h = s1a.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.006; const [xa, xb, xc, xd] = ctx; const V = warp.v;
			xa.fillStyle = V > 3 ? `rgba(0,0,0,${Math.max(0.03, 0.2 - V * 0.015)})` : 'rgba(0,0,0,0.2)'; xa.fillRect(0, 0, w, h);
			const cx = w / 2, cy = h / 2, mx_pos = (mx / w - 0.5) * 80, my_pos = (my / h - 0.5) * 80;
			stars.forEach(s => { s.z -= V * 0.015; if (s.z <= 0.05) { s.z = 5; s.x = (Math.random() - 0.5) * 3; s.y = (Math.random() - 0.5) * 3; } const sx = cx + mx_pos * 0.3 + (s.x / s.z) * w * 0.5, sy = cy + my_pos * 0.3 + (s.y / s.z) * h * 0.5; if (sx < -10 || sx > w + 10 || sy < -10 || sy > h + 10) return; const bright = 1 - s.z / 5; if (V > 2) { const pz = s.z + V * 0.015, px = cx + mx_pos * 0.3 + (s.x / pz) * w * 0.5, py = cy + my_pos * 0.3 + (s.y / pz) * h * 0.5; xa.beginPath(); xa.moveTo(px, py); xa.lineTo(sx, sy); xa.strokeStyle = `hsla(${s.hue},70%,${60 + bright * 40}%,${bright * 0.9})`; xa.lineWidth = s.size * (V > 5 ? 1.5 : 0.8); xa.stroke(); xa.beginPath(); xa.arc(sx, sy, s.size * 0.6, 0, Math.PI * 2); xa.fillStyle = `rgba(255,255,255,${bright * 0.9})`; xa.fill(); } else { xa.beginPath(); xa.arc(sx, sy, s.size * bright, 0, Math.PI * 2); xa.fillStyle = `hsla(${s.hue},60%,${70 + bright * 30}%,${bright * 0.8})`; xa.fill(); } });
			xb.clearRect(0, 0, w, h); const voidR = 55 + Math.sin(t * 3) * 3; const vGrd = xb.createRadialGradient(cx, cy, 0, cx, cy, voidR + 30); vGrd.addColorStop(0, 'rgba(0,0,0,1)'); vGrd.addColorStop(0.6, 'rgba(0,0,0,0.98)'); vGrd.addColorStop(0.85, 'rgba(0,0,0,0.7)'); vGrd.addColorStop(1, 'rgba(0,0,0,0)'); xb.fillStyle = vGrd; xb.beginPath(); xb.arc(cx, cy, voidR + 30, 0, Math.PI * 2); xb.fill();
			xb.beginPath(); xb.arc(cx, cy, voidR + 2, 0, Math.PI * 2); xb.strokeStyle = `rgba(255,200,100,${0.6 + Math.sin(t * 5) * 0.15})`; xb.lineWidth = 3; xb.shadowBlur = 30; xb.shadowColor = 'rgba(255,150,50,0.8)'; xb.stroke();
			xb.beginPath(); xb.arc(cx, cy, voidR + 8, 0, Math.PI * 2); xb.strokeStyle = `rgba(255,100,30,${0.25 + Math.sin(t * 4) * 0.1})`; xb.lineWidth = 1.5; xb.stroke(); xb.shadowBlur = 0;
			xc.clearRect(0, 0, w, h); xc.globalCompositeOperation = 'lighter'; disk.sort((a, b) => Math.sin(a.angle) * a.dist - Math.sin(b.angle) * b.dist);
			disk.forEach(p => { p.angle += p.speed * 0.008; const x3 = Math.cos(p.angle) * p.dist, y3 = Math.sin(p.angle) * p.dist * p.tilt, z3 = Math.sin(p.angle) * p.dist, px = cx + x3, py = cy + y3; if (Math.sqrt(x3 * x3 + y3 * y3) < voidR - 5) return; const heat = Math.pow(Math.max(0, 1 - p.dist / 300), 1.5), depthFade = (z3 + 300) / 600, g = Math.floor(80 + heat * 170), b = Math.floor(heat * 40), alpha = p.bright * depthFade * (0.3 + heat * 0.7); xc.beginPath(); xc.arc(px, py, p.size * (0.6 + heat * 1.2), 0, Math.PI * 2); xc.fillStyle = `rgba(255,${g},${b},${alpha})`; if (heat > 0.4) { xc.shadowBlur = p.size * 6; xc.shadowColor = `rgba(255,${g},0,${alpha * 0.6})`; } xc.fill(); xc.shadowBlur = 0; });
			xc.globalCompositeOperation = 'source-over'; xd.clearRect(0, 0, w, h); xd.globalCompositeOperation = 'lighter'; const hGrd = xd.createRadialGradient(cx, cy, voidR, cx, cy, 350); hGrd.addColorStop(0, `rgba(255,120,20,${0.08 + Math.sin(t * 2) * 0.03})`); hGrd.addColorStop(0.4, 'rgba(255,60,10,0.04)'); hGrd.addColorStop(1, 'transparent'); xd.save(); xd.translate(cx, cy); xd.scale(1, 0.35); xd.translate(-cx, -cy); xd.fillStyle = hGrd; xd.fillRect(0, 0, w, h); xd.restore(); xd.globalCompositeOperation = 'source-over';
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s1', start: 'top 60%', onEnter: () => { on = true; const tl = gsap.timeline(); tl.to(warp, { v: 0.5, duration: 2, ease: 'none' }, 0); tl.to(warp, { v: 20, duration: 2.5, ease: 'power3.in' }, 2); tl.to('#s1f', { opacity: 1, duration: 0.06 }, 4.2); tl.to('#s1f', { opacity: 0, duration: 2, ease: 'power3.out' }, 4.3); tl.to(warp, { v: 0.3, duration: 3, ease: 'power3.out' }, 4.5); tl.to('#s1t', { opacity: 1, scale: 1, filter: 'blur(0px)', duration: 0.5, ease: 'power4.out' }, 4.3); tl.from('#s1t', { scale: 4, filter: 'blur(40px)', duration: 0.5, ease: 'power4.out' }, 4.3); tl.add(() => shake(document.getElementById('s1-content')!, 50, 0.8), 4.3); tl.to('#s1s', { opacity: 1, duration: 1.2 }, 5); } }); triggers.push(trigger);
	}

	function initInferno() {
		const cv = [s2a, s2b, s2c];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s2a.width = s2a.parentElement!.offsetWidth, h = s2a.height = s2a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const intensity = { v: 0 };
		const fire: { x: number; y: number; vx: number; vy: number; life: number; decay: number; size: number; hue: number }[] = [];
		const embers: { x: number; y: number; vx: number; vy: number; size: number; life: number; hue: number }[] = [];
		for (let i = 0; i < 500; i++) embers.push({ x: Math.random() * w, y: h + 20 + Math.random() * 200, vx: (Math.random() - 0.5) * 3, vy: -Math.random() * 4 - 1, size: Math.random() * 4 + 1, life: Math.random(), hue: 10 + Math.random() * 40 });
		const cracks: { pts: { x: number; y: number }[]; width: number; phase: number }[] = [];
		for (let i = 0; i < 12; i++) { const pts: { x: number; y: number }[] = []; let cx_pos = Math.random() * w, cy_pos = h * 0.9; for (let j = 0; j < 15; j++) { cx_pos += (Math.random() - 0.5) * 100; cy_pos -= Math.random() * 50 + 15; pts.push({ x: cx_pos / w, y: cy_pos / h }); } cracks.push({ pts, width: Math.random() * 4 + 2, phase: Math.random() * Math.PI * 2 }); }
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s2a.width; h = s2a.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.008; const I = intensity.v, [xa, xb, xc] = ctx;
			xa.globalCompositeOperation = 'source-over'; xa.fillStyle = `rgba(2,0,0,${0.12 - I * 0.03})`; xa.fillRect(0, 0, w, h); xa.globalCompositeOperation = 'lighter';
			const spawnRate = Math.floor(I * 60); for (let i = 0; i < spawnRate && fire.length < 5000; i++) { const fromMouse = Math.random() > 0.6, bx = fromMouse ? mx + (Math.random() - 0.5) * 80 : w / 2 + (Math.random() - 0.5) * w * 0.5, by = fromMouse ? my : h + 5; fire.push({ x: bx, y: by, vx: (Math.random() - 0.5) * 4 + (fromMouse ? (Math.random() - 0.5) * 6 : 0), vy: fromMouse ? -Math.random() * 10 - 5 : -Math.random() * 10 - 4, life: 1, decay: 0.008 + Math.random() * 0.015, size: Math.random() * 12 + 4, hue: 5 + Math.random() * 45 }); }
			for (let i = fire.length - 1; i >= 0; i--) { const p = fire[i]; p.vy -= 0.12; p.vx += Math.sin(t * 8 + p.x * 0.005) * 0.8; p.vx *= 0.98; p.x += p.vx; p.y += p.vy; p.life -= p.decay; if (p.life <= 0 || p.y < -100) { fire.splice(i, 1); continue; } const fadeHue = p.hue + (1 - p.life) * 20, lum = 30 + p.life * 40, a = p.life * I * 0.7, s = p.size * p.life; xa.beginPath(); xa.arc(p.x, p.y, s, 0, Math.PI * 2); xa.fillStyle = `hsla(${fadeHue},100%,${lum}%,${a})`; xa.fill(); if (s > 5) { xa.beginPath(); xa.arc(p.x, p.y, s * 0.35, 0, Math.PI * 2); xa.fillStyle = `hsla(${fadeHue + 10},100%,${lum + 25}%,${a * 0.8})`; xa.fill(); } }
			xa.globalCompositeOperation = 'source-over'; xb.clearRect(0, 0, w, h); xb.globalCompositeOperation = 'lighter';
			embers.forEach(e => { e.x += e.vx + Math.sin(t * 3 + e.x * 0.003) * 1; e.y += e.vy * I; e.life -= 0.003; if (e.life <= 0 || e.y < -20) { e.y = h + 20; e.x = Math.random() * w; e.life = 1; e.vy = -Math.random() * 4 - 1; } const flicker = 0.4 + Math.sin(t * 20 + e.x * 2) * 0.6; xb.beginPath(); xb.arc(e.x, e.y, e.size * e.life, 0, Math.PI * 2); xb.fillStyle = `hsla(${e.hue},100%,60%,${e.life * I * flicker * 0.8})`; xb.shadowBlur = e.size * 5; xb.shadowColor = `hsla(${e.hue},100%,50%,${e.life * I * 0.5})`; xb.fill(); xb.shadowBlur = 0; if (I > 0.5) { xb.beginPath(); xb.moveTo(e.x, e.y); xb.lineTo(e.x - e.vx * 3, e.y - e.vy * I * 3); xb.strokeStyle = `hsla(${e.hue},100%,60%,${e.life * I * 0.3})`; xb.lineWidth = e.size * 0.5; xb.stroke(); } });
			xb.globalCompositeOperation = 'source-over'; xc.clearRect(0, 0, w, h);
			cracks.forEach(cr => { const glow = 0.4 + Math.sin(t * 2.5 + cr.phase) * 0.3; xc.beginPath(); cr.pts.forEach((p, i) => { const px = p.x * w + Math.sin(t * 4 + i) * 4, py = p.y * h; if (i === 0) xc.moveTo(px, py); else xc.lineTo(px, py); }); xc.strokeStyle = `rgba(255,60,0,${glow * I * 0.7})`; xc.lineWidth = cr.width * 3; xc.shadowBlur = 30; xc.shadowColor = `rgba(255,40,0,${glow * I * 0.6})`; xc.stroke(); xc.beginPath(); cr.pts.forEach((p, i) => { const px = p.x * w + Math.sin(t * 4 + i) * 4, py = p.y * h; if (i === 0) xc.moveTo(px, py); else xc.lineTo(px, py); }); xc.strokeStyle = `rgba(255,200,50,${glow * I * 0.8})`; xc.lineWidth = cr.width * 0.6; xc.stroke(); xc.shadowBlur = 0; xc.beginPath(); cr.pts.forEach((p, i) => { const px = p.x * w + Math.sin(t * 4 + i) * 4, py = p.y * h; if (i === 0) xc.moveTo(px, py); else xc.lineTo(px, py); }); xc.strokeStyle = `rgba(255,255,200,${glow * I * 0.4})`; xc.lineWidth = cr.width * 0.2; xc.stroke(); });
			const magmaGrd = xc.createLinearGradient(0, h * 0.7, 0, h); magmaGrd.addColorStop(0, 'transparent'); magmaGrd.addColorStop(0.5, `rgba(255,40,0,${I * 0.15})`); magmaGrd.addColorStop(0.8, `rgba(255,80,0,${I * 0.25})`); magmaGrd.addColorStop(1, `rgba(255,120,20,${I * 0.3})`); xc.fillStyle = magmaGrd; xc.fillRect(0, h * 0.7, w, h * 0.3);
			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s2', start: 'top 60%', onEnter: () => { on = true; const tl = gsap.timeline(); tl.to(intensity, { v: 1, duration: 4, ease: 'power2.in' }, 0); tl.to('#s2f', { opacity: 0.7, duration: 0.1 }, 3.5); tl.to('#s2f', { opacity: 0, duration: 2 }, 3.6); tl.to('#s2t', { opacity: 1, duration: 0.5 }, 3.8); tl.from('#s2t', { scale: 3, filter: 'blur(20px)', duration: 0.5, ease: 'power4.out' }, 3.8); tl.add(() => shake(document.getElementById('s2-content')!, 30, 0.6), 3.8); tl.to('#s2s', { opacity: 1, duration: 1.2 }, 4.5); } }); triggers.push(trigger);
	}

	function initVoltage() {
		const cv = [s3a, s3b, s3c];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s3a.width = s3a.parentElement!.offsetWidth, h = s3a.height = s3a.parentElement!.offsetHeight;
		let t = 0, on = false;
		const power = { v: 0 };
		const ions: { x: number; y: number; vx: number; vy: number; size: number; charge: number }[] = [];
		for (let i = 0; i < 800; i++) ions.push({ x: Math.random() * w, y: Math.random() * h, vx: 0, vy: 0, size: Math.random() * 2.5 + 0.5, charge: Math.random() });
		function bolt(ctx: CanvasRenderingContext2D, x1: number, y1: number, x2: number, y2: number, gen: number, maxGen: number, mainAlpha: number) {
			if (gen > maxGen) return; const segs = 6 + Math.floor(Math.random() * 6); const pts: { x: number; y: number }[] = [{ x: x1, y: y1 }]; const dx = x2 - x1, dy = y2 - y1, len = Math.sqrt(dx * dx + dy * dy), nx = -dy / len, ny = dx / len;
			for (let i = 1; i < segs; i++) { const ti = i / segs, offset = (Math.random() - 0.5) * len * 0.2 / (gen + 1); pts.push({ x: x1 + dx * ti + nx * offset, y: y1 + dy * ti + ny * offset }); } pts.push({ x: x2, y: y2 });
			const alpha = mainAlpha / (gen * 0.7 + 1); ctx.beginPath(); pts.forEach((p, i) => { if (i === 0) ctx.moveTo(p.x, p.y); else ctx.lineTo(p.x, p.y); }); ctx.strokeStyle = `rgba(180,200,255,${alpha})`; ctx.lineWidth = Math.max(0.5, 4 / (gen + 1)); ctx.shadowBlur = 20 / (gen + 1); ctx.shadowColor = `rgba(100,150,255,${alpha})`; ctx.stroke();
			ctx.beginPath(); pts.forEach((p, i) => { if (i === 0) ctx.moveTo(p.x, p.y); else ctx.lineTo(p.x, p.y); }); ctx.strokeStyle = `rgba(255,255,255,${alpha * 0.6})`; ctx.lineWidth = Math.max(0.3, 2 / (gen + 1)); ctx.stroke(); ctx.shadowBlur = 0;
			if (gen < maxGen - 1) { const branchCount = gen === 0 ? 2 : 1; for (let b = 0; b < branchCount; b++) { if (Math.random() > 0.5) continue; const bi = Math.floor(segs * 0.2 + Math.random() * segs * 0.6), bp = pts[bi]; if (!bp) continue; const ba = Math.atan2(dy, dx) + (Math.random() - 0.5) * 2, bl = len * (0.2 + Math.random() * 0.3) / (gen + 1); bolt(ctx, bp.x, bp.y, bp.x + Math.cos(ba) * bl, bp.y + Math.sin(ba) * bl, gen + 1, maxGen, mainAlpha); } }
		}
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s3a.width; h = s3a.height; }
		window.addEventListener('resize', resize);
		function draw() {
			if (!on) { rafIds.push(requestAnimationFrame(draw)); return; }
			t += 0.01; const P = power.v, [xa, xb, xc] = ctx, cx = w / 2, cy = h / 2;
			xa.fillStyle = `rgba(2,0,8,${0.2 - P * 0.05})`; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xc.clearRect(0, 0, w, h);
			xa.globalCompositeOperation = 'lighter'; const coreGrd = xa.createRadialGradient(cx, cy, 0, cx, cy, 100 + P * 50); coreGrd.addColorStop(0, `rgba(120,100,255,${P * 0.2})`); coreGrd.addColorStop(0.3, `rgba(80,60,200,${P * 0.1})`); coreGrd.addColorStop(1, 'transparent'); xa.fillStyle = coreGrd; xa.fillRect(0, 0, w, h);
			const mGrd = xa.createRadialGradient(mx, my, 0, mx, my, 80); mGrd.addColorStop(0, `rgba(150,120,255,${P * 0.15})`); mGrd.addColorStop(1, 'transparent'); xa.fillStyle = mGrd; xa.fillRect(mx - 80, my - 80, 160, 160); xa.globalCompositeOperation = 'source-over';
			xb.beginPath(); xb.arc(cx, cy, 18 + Math.sin(t * 8) * 5, 0, Math.PI * 2); xb.fillStyle = `rgba(200,180,255,${P * 0.7 + Math.sin(t * 6) * 0.1})`; xb.shadowBlur = 60; xb.shadowColor = `rgba(100,80,255,${P * 0.8})`; xb.fill(); xb.beginPath(); xb.arc(cx, cy, 8 + Math.sin(t * 10) * 2, 0, Math.PI * 2); xb.fillStyle = `rgba(255,255,255,${P * 0.8})`; xb.fill(); xb.shadowBlur = 0;
			if (P > 0.2) { const boltCount = Math.floor(P * 8); for (let i = 0; i < boltCount; i++) { if (i === 0 || i === 1) bolt(xb, cx, cy, mx + (Math.random() - 0.5) * 30, my + (Math.random() - 0.5) * 30, 0, 3, P * 0.9); else { const edge = Math.floor(Math.random() * 4); let ex, ey; if (edge === 0) { ex = Math.random() * w; ey = 0; } else if (edge === 1) { ex = w; ey = Math.random() * h; } else if (edge === 2) { ex = Math.random() * w; ey = h; } else { ex = 0; ey = Math.random() * h; } bolt(xb, cx, cy, ex, ey, 0, 2, P * 0.5); } } if (Math.random() > 0.5) bolt(xb, mx, my, mx + (Math.random() - 0.5) * 300, my + (Math.random() - 0.5) * 300, 0, 2, P * 0.4); }
			xc.globalCompositeOperation = 'lighter'; ions.forEach(ion => { const dcx = cx - ion.x, dcy = cy - ion.y, dc = Math.sqrt(dcx * dcx + dcy * dcy), dmx = mx - ion.x, dmy = my - ion.y, dm = Math.sqrt(dmx * dmx + dmy * dmy); if (dc < 300) { ion.vx += (dcx / dc) * P * 0.8; ion.vy += (dcy / dc) * P * 0.8; } if (dm < 200) { ion.vx += (dmx / dm) * P * 1.2; ion.vy += (dmy / dm) * P * 1.2; } ion.vx *= 0.94; ion.vy *= 0.94; ion.x += ion.vx; ion.y += ion.vy; if (ion.x < -50 || ion.x > w + 50 || ion.y < -50 || ion.y > h + 50) { ion.x = Math.random() * w; ion.y = Math.random() * h; ion.vx = 0; ion.vy = 0; } const speed = Math.sqrt(ion.vx * ion.vx + ion.vy * ion.vy), a = Math.min(1, (0.15 + speed * 0.08) * P); xc.beginPath(); xc.arc(ion.x, ion.y, ion.size * (1 + speed * 0.1), 0, Math.PI * 2); xc.fillStyle = `rgba(150,130,255,${a})`; if (speed > 3) { xc.shadowBlur = 8; xc.shadowColor = `rgba(120,100,255,${a * 0.5})`; } xc.fill(); xc.shadowBlur = 0; if (speed > 2 && P > 0.5) { xc.beginPath(); xc.moveTo(ion.x, ion.y); xc.lineTo(ion.x - ion.vx * 2, ion.y - ion.vy * 2); xc.strokeStyle = `rgba(150,130,255,${a * 0.4})`; xc.lineWidth = ion.size * 0.5; xc.stroke(); } });
			xc.globalCompositeOperation = 'source-over'; rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s3', start: 'top 60%', onEnter: () => { on = true; const tl = gsap.timeline(); tl.to(power, { v: 1, duration: 3, ease: 'power2.in' }, 0); tl.to('#s3f', { opacity: 0.4, duration: 0.05 }, 2.5); tl.to('#s3f', { opacity: 0, duration: 0.8 }, 2.55); tl.add(() => shake(document.getElementById('s3-content')!, 20, 0.4), 2.5); tl.to('#s3t', { opacity: 1, duration: 0.4 }, 2.8); tl.from('#s3t', { scale: 2, filter: 'blur(15px)', duration: 0.4, ease: 'power4.out' }, 2.8); tl.to('#s3s', { opacity: 1, duration: 1 }, 3.5); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; prevMx = mx; prevMy = my; rafIds.push(requestAnimationFrame(updateMouseVelocity)); initSingularity(); initInferno(); initVoltage(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="s1" style="background:#000308">
	<canvas bind:this={s1a}></canvas>
	<canvas bind:this={s1b} class="c2"></canvas>
	<canvas bind:this={s1c} class="c3"></canvas>
	<canvas bind:this={s1d} class="c4"></canvas>
	<div class="flash" id="s1f" style="background:#fff"></div>
	<div class="ct" id="s1-content">
		<h1 class="t1" id="s1t">SINGULARITY</h1>
		<p class="t2" id="s1s">NOTHING ESCAPES. NOT EVEN LIGHT.</p>
	</div>
	<span class="lbl">01 — EVENT HORIZON</span>
	<span class="idx">01</span>
</section>
<div class="sp"></div>

<section class="hero" id="s2" style="background:#0a0005">
	<canvas bind:this={s2a}></canvas>
	<canvas bind:this={s2b} class="c2"></canvas>
	<canvas bind:this={s2c} class="c3"></canvas>
	<div class="flash" id="s2f" style="background:#ff3300"></div>
	<div class="ct" id="s2-content">
		<h1 class="t1" id="s2t">INFERNO</h1>
		<p class="t2" id="s2s">THE CORE MELTS. THE WORLD BURNS.</p>
	</div>
	<span class="lbl">02 — MAGMA</span>
	<span class="idx">02</span>
</section>
<div class="sp"></div>

<section class="hero" id="s3" style="background:#000208">
	<canvas bind:this={s3a}></canvas>
	<canvas bind:this={s3b} class="c2"></canvas>
	<canvas bind:this={s3c} class="c3"></canvas>
	<div class="flash" id="s3f" style="background:#8b5cf6"></div>
	<div class="ct" id="s3-content">
		<h1 class="t1" id="s3t">VOLTAGE</h1>
		<p class="t2" id="s3s">THE GOD OF ELECTRICITY AWAKENS.</p>
	</div>
	<span class="lbl">03 — TESLA COIL</span>
	<span class="idx">03</span>
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
	.sp { height: 2px; background: #000; }
	#s1 { background: #000308; } #s2 { background: #0a0005; } #s3 { background: #000208; }
</style>
