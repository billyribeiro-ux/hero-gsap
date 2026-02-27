<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	// Mouse tracking
	let mx = 0;
	let my = 0;
	let prevMx = 0;
	let prevMy = 0;
	let mvx = 0;
	let mvy = 0;

	// Canvas refs - 5 sections
	let s1a: HTMLCanvasElement, s1b: HTMLCanvasElement, s1c: HTMLCanvasElement, s1d: HTMLCanvasElement;
	let s2a: HTMLCanvasElement, s2b: HTMLCanvasElement, s2c: HTMLCanvasElement;
	let s3a: HTMLCanvasElement, s3b: HTMLCanvasElement, s3c: HTMLCanvasElement;
	let s4a: HTMLCanvasElement, s4b: HTMLCanvasElement, s4c: HTMLCanvasElement, s4d: HTMLCanvasElement;
	let s5a: HTMLCanvasElement, s5b: HTMLCanvasElement, s5c: HTMLCanvasElement, s5d: HTMLCanvasElement;

	// Animation state
	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];
	let chartInterval: number;

	function handleMouseMove(e: MouseEvent) {
		mx = e.clientX;
		my = e.clientY;
	}

	function updateMouseVelocity() {
		mvx = mx - prevMx;
		mvy = my - prevMy;
		prevMx = mx;
		prevMy = my;
		rafIds.push(requestAnimationFrame(updateMouseVelocity));
	}

	function shake(el: HTMLElement, intensity = 20, dur = 0.5) {
		const s = Math.floor(dur / 0.016);
		for (let i = 0; i < s; i++) {
			const f = Math.pow(1 - i / s, 2);
			gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.6, delay: i * 0.016 });
		}
		gsap.set(el, { x: 0, y: 0, delay: s * 0.016 });
	}

	/* ══════════════════════════════════════════════════════════════
	   1. SUPERMASSIVE BLACK HOLE
	   ══════════════════════════════════════════════════════════════ */
	function initSingularity() {
		const cv = [s1a, s1b, s1c, s1d];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s1a.width = s1a.parentElement!.offsetWidth;
		let h = s1a.height = s1a.parentElement!.offsetHeight;
		let t = 0;
		let on = false;
		const warp = { v: 0 };

		const STARS = 4000;
		const stars: { x: number; y: number; z: number; size: number; hue: number }[] = [];
		for (let i = 0; i < STARS; i++) {
			stars.push({
				x: (Math.random() - 0.5) * 3,
				y: (Math.random() - 0.5) * 3,
				z: Math.random() * 5 + 0.5,
				size: Math.random() * 1.8 + 0.5,
				hue: 200 + Math.random() * 60
			});
		}

		const DISK = 2000;
		const disk: { dist: number; angle: number; speed: number; tilt: number; size: number; hue: number; bright: number }[] = [];
		for (let i = 0; i < DISK; i++) {
			const d = 40 + Math.pow(Math.random(), 0.5) * 280;
			const a = Math.random() * Math.PI * 2;
			const speed = 1.2 / Math.sqrt(d / 60);
			disk.push({
				dist: d,
				angle: a,
				speed: speed * (0.7 + Math.random() * 0.6),
				tilt: 0.28 + Math.random() * 0.08,
				size: Math.random() * 3 + 1,
				hue: 15 + Math.random() * 35,
				bright: 0.5 + Math.random() * 0.5
			});
		}

		function resize() {
			cv.forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = s1a.width;
			h = s1a.height;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!on) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.006;
			const [xa, xb, xc, xd] = ctx;
			const V = warp.v;

			xa.fillStyle = V > 3 ? `rgba(0,0,0,${Math.max(0.03, 0.2 - V * 0.015)})` : 'rgba(0,0,0,0.2)';
			xa.fillRect(0, 0, w, h);

			const cx = w / 2;
			const cy = h / 2;
			const mx_pos = (mx / w - 0.5) * 80;
			const my_pos = (my / h - 0.5) * 80;

			stars.forEach(s => {
				s.z -= V * 0.015;
				if (s.z <= 0.05) {
					s.z = 5;
					s.x = (Math.random() - 0.5) * 3;
					s.y = (Math.random() - 0.5) * 3;
				}
				const sx = cx + mx_pos * 0.3 + (s.x / s.z) * w * 0.5;
				const sy = cy + my_pos * 0.3 + (s.y / s.z) * h * 0.5;
				if (sx < -10 || sx > w + 10 || sy < -10 || sy > h + 10) return;
				const bright = 1 - s.z / 5;

				if (V > 2) {
					const pz = s.z + V * 0.015;
					const px = cx + mx_pos * 0.3 + (s.x / pz) * w * 0.5;
					const py = cy + my_pos * 0.3 + (s.y / pz) * h * 0.5;
					xa.beginPath();
					xa.moveTo(px, py);
					xa.lineTo(sx, sy);
					xa.strokeStyle = `hsla(${s.hue},70%,${60 + bright * 40}%,${bright * 0.9})`;
					xa.lineWidth = s.size * (V > 5 ? 1.5 : 0.8);
					xa.stroke();
					xa.beginPath();
					xa.arc(sx, sy, s.size * 0.6, 0, Math.PI * 2);
					xa.fillStyle = `rgba(255,255,255,${bright * 0.9})`;
					xa.fill();
				} else {
					xa.beginPath();
					xa.arc(sx, sy, s.size * bright, 0, Math.PI * 2);
					xa.fillStyle = `hsla(${s.hue},60%,${70 + bright * 30}%,${bright * 0.8})`;
					xa.fill();
				}
			});

			xb.clearRect(0, 0, w, h);
			const voidR = 55 + Math.sin(t * 3) * 3;
			const vGrd = xb.createRadialGradient(cx, cy, 0, cx, cy, voidR + 30);
			vGrd.addColorStop(0, 'rgba(0,0,0,1)');
			vGrd.addColorStop(0.6, 'rgba(0,0,0,0.98)');
			vGrd.addColorStop(0.85, 'rgba(0,0,0,0.7)');
			vGrd.addColorStop(1, 'rgba(0,0,0,0)');
			xb.fillStyle = vGrd;
			xb.beginPath();
			xb.arc(cx, cy, voidR + 30, 0, Math.PI * 2);
			xb.fill();

			xb.beginPath();
			xb.arc(cx, cy, voidR + 2, 0, Math.PI * 2);
			xb.strokeStyle = `rgba(255,200,100,${0.6 + Math.sin(t * 5) * 0.15})`;
			xb.lineWidth = 3;
			xb.shadowBlur = 30;
			xb.shadowColor = 'rgba(255,150,50,0.8)';
			xb.stroke();

			xb.beginPath();
			xb.arc(cx, cy, voidR + 8, 0, Math.PI * 2);
			xb.strokeStyle = `rgba(255,100,30,${0.25 + Math.sin(t * 4) * 0.1})`;
			xb.lineWidth = 1.5;
			xb.stroke();
			xb.shadowBlur = 0;

			xc.clearRect(0, 0, w, h);
			xc.globalCompositeOperation = 'lighter';

			disk.sort((a, b) => {
				const az = Math.sin(a.angle) * a.dist;
				const bz = Math.sin(b.angle) * b.dist;
				return az - bz;
			});

			disk.forEach(p => {
				p.angle += p.speed * 0.008;
				const x3 = Math.cos(p.angle) * p.dist;
				const y3 = Math.sin(p.angle) * p.dist * p.tilt;
				const z3 = Math.sin(p.angle) * p.dist;
				const px = cx + x3;
				const py = cy + y3;
				const distC = Math.sqrt(x3 * x3 + y3 * y3);
				if (distC < voidR - 5) return;

				const heat = Math.pow(Math.max(0, 1 - p.dist / 300), 1.5);
				const depthFade = (z3 + 300) / 600;
				const r = 255;
				const g = Math.floor(80 + heat * 170);
				const b = Math.floor(heat * 40);
				const alpha = p.bright * depthFade * (0.3 + heat * 0.7);

				xc.beginPath();
				xc.arc(px, py, p.size * (0.6 + heat * 1.2), 0, Math.PI * 2);
				xc.fillStyle = `rgba(${r},${g},${b},${alpha})`;
				if (heat > 0.4) {
					xc.shadowBlur = p.size * 6;
					xc.shadowColor = `rgba(255,${g},0,${alpha * 0.6})`;
				}
				xc.fill();
				xc.shadowBlur = 0;
			});
			xc.globalCompositeOperation = 'source-over';

			xd.clearRect(0, 0, w, h);
			xd.globalCompositeOperation = 'lighter';
			const hGrd = xd.createRadialGradient(cx, cy, voidR, cx, cy, 350);
			hGrd.addColorStop(0, `rgba(255,120,20,${0.08 + Math.sin(t * 2) * 0.03})`);
			hGrd.addColorStop(0.4, 'rgba(255,60,10,0.04)');
			hGrd.addColorStop(1, 'transparent');
			xd.save();
			xd.translate(cx, cy);
			xd.scale(1, 0.35);
			xd.translate(-cx, -cy);
			xd.fillStyle = hGrd;
			xd.fillRect(0, 0, w, h);
			xd.restore();
			xd.globalCompositeOperation = 'source-over';

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s1',
			start: 'top 60%',
			onEnter: () => {
				on = true;
				const tl = gsap.timeline();
				tl.to(warp, { v: 0.5, duration: 2, ease: 'none' }, 0);
				tl.to(warp, { v: 20, duration: 2.5, ease: 'power3.in' }, 2);
				tl.to('#s1f', { opacity: 1, duration: 0.06 }, 4.2);
				tl.to('#s1f', { opacity: 0, duration: 2, ease: 'power3.out' }, 4.3);
				tl.to(warp, { v: 0.3, duration: 3, ease: 'power3.out' }, 4.5);
				tl.to('#s1t', { opacity: 1, scale: 1, filter: 'blur(0px)', duration: 0.5, ease: 'power4.out' }, 4.3);
				tl.from('#s1t', { scale: 4, filter: 'blur(40px)', duration: 0.5, ease: 'power4.out' }, 4.3);
				tl.add(() => shake(document.getElementById('s1-content')!, 50, 0.8), 4.3);
				tl.to('#s1s', { opacity: 1, duration: 1.2 }, 5);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   2. INFERNO — Real fire
	   ══════════════════════════════════════════════════════════════ */
	function initInferno() {
		const cv = [s2a, s2b, s2c];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s2a.width = s2a.parentElement!.offsetWidth;
		let h = s2a.height = s2a.parentElement!.offsetHeight;
		let t = 0;
		let on = false;
		const intensity = { v: 0 };

		const MAX_FIRE = 5000;
		const fire: { x: number; y: number; vx: number; vy: number; life: number; decay: number; size: number; hue: number }[] = [];

		const embers: { x: number; y: number; vx: number; vy: number; size: number; life: number; hue: number }[] = [];
		for (let i = 0; i < 500; i++) {
			embers.push({
				x: Math.random() * w,
				y: h + 20 + Math.random() * 200,
				vx: (Math.random() - 0.5) * 3,
				vy: -Math.random() * 4 - 1,
				size: Math.random() * 4 + 1,
				life: Math.random(),
				hue: 10 + Math.random() * 40
			});
		}

		const cracks: { pts: { x: number; y: number }[]; width: number; phase: number }[] = [];
		for (let i = 0; i < 12; i++) {
			const pts: { x: number; y: number }[] = [];
			let cx_pos = Math.random() * w;
			let cy_pos = h * 0.9;
			for (let j = 0; j < 15; j++) {
				cx_pos += (Math.random() - 0.5) * 100;
				cy_pos -= Math.random() * 50 + 15;
				pts.push({ x: cx_pos / w, y: cy_pos / h });
			}
			cracks.push({ pts, width: Math.random() * 4 + 2, phase: Math.random() * Math.PI * 2 });
		}

		function resize() {
			cv.forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = s2a.width;
			h = s2a.height;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!on) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.008;
			const I = intensity.v;
			const [xa, xb, xc] = ctx;

			xa.globalCompositeOperation = 'source-over';
			xa.fillStyle = `rgba(2,0,0,${0.12 - I * 0.03})`;
			xa.fillRect(0, 0, w, h);
			xa.globalCompositeOperation = 'lighter';

			const spawnRate = Math.floor(I * 60);
			for (let i = 0; i < spawnRate && fire.length < MAX_FIRE; i++) {
				const fromMouse = Math.random() > 0.6;
				const bx = fromMouse ? mx + (Math.random() - 0.5) * 80 : w / 2 + (Math.random() - 0.5) * w * 0.5;
				const by = fromMouse ? my : h + 5;
				fire.push({
					x: bx,
					y: by,
					vx: (Math.random() - 0.5) * 4 + (fromMouse ? (Math.random() - 0.5) * 6 : 0),
					vy: fromMouse ? -Math.random() * 10 - 5 : -Math.random() * 10 - 4,
					life: 1,
					decay: 0.008 + Math.random() * 0.015,
					size: Math.random() * 12 + 4,
					hue: 5 + Math.random() * 45
				});
			}

			for (let i = fire.length - 1; i >= 0; i--) {
				const p = fire[i];
				p.vy -= 0.12;
				p.vx += Math.sin(t * 8 + p.x * 0.005) * 0.8;
				p.vx *= 0.98;
				p.x += p.vx;
				p.y += p.vy;
				p.life -= p.decay;

				if (p.life <= 0 || p.y < -100) {
					fire.splice(i, 1);
					continue;
				}

				const fadeHue = p.hue + (1 - p.life) * 20;
				const lum = 30 + p.life * 40;
				const sat = 100;
				const a = p.life * I * 0.7;
				const s = p.size * p.life;

				xa.beginPath();
				xa.arc(p.x, p.y, s, 0, Math.PI * 2);
				xa.fillStyle = `hsla(${fadeHue},${sat}%,${lum}%,${a})`;
				xa.fill();

				if (s > 5) {
					xa.beginPath();
					xa.arc(p.x, p.y, s * 0.35, 0, Math.PI * 2);
					xa.fillStyle = `hsla(${fadeHue + 10},100%,${lum + 25}%,${a * 0.8})`;
					xa.fill();
				}
			}
			xa.globalCompositeOperation = 'source-over';

			xb.clearRect(0, 0, w, h);
			xb.globalCompositeOperation = 'lighter';
			embers.forEach(e => {
				e.x += e.vx + Math.sin(t * 3 + e.x * 0.003) * 1;
				e.y += e.vy * I;
				e.life -= 0.003;
				if (e.life <= 0 || e.y < -20) {
					e.y = h + 20;
					e.x = Math.random() * w;
					e.life = 1;
					e.vy = -Math.random() * 4 - 1;
				}
				const flicker = 0.4 + Math.sin(t * 20 + e.x * 2) * 0.6;
				xb.beginPath();
				xb.arc(e.x, e.y, e.size * e.life, 0, Math.PI * 2);
				xb.fillStyle = `hsla(${e.hue},100%,60%,${e.life * I * flicker * 0.8})`;
				xb.shadowBlur = e.size * 5;
				xb.shadowColor = `hsla(${e.hue},100%,50%,${e.life * I * 0.5})`;
				xb.fill();
				xb.shadowBlur = 0;

				if (I > 0.5) {
					xb.beginPath();
					xb.moveTo(e.x, e.y);
					xb.lineTo(e.x - e.vx * 3, e.y - e.vy * I * 3);
					xb.strokeStyle = `hsla(${e.hue},100%,60%,${e.life * I * 0.3})`;
					xb.lineWidth = e.size * 0.5;
					xb.stroke();
				}
			});
			xb.globalCompositeOperation = 'source-over';

			xc.clearRect(0, 0, w, h);
			cracks.forEach(cr => {
				const glow = 0.4 + Math.sin(t * 2.5 + cr.phase) * 0.3;
				xc.beginPath();
				cr.pts.forEach((p, i) => {
					const px = p.x * w + Math.sin(t * 4 + i) * 4;
					const py = p.y * h;
					if (i === 0) xc.moveTo(px, py);
					else xc.lineTo(px, py);
				});
				xc.strokeStyle = `rgba(255,60,0,${glow * I * 0.7})`;
				xc.lineWidth = cr.width * 3;
				xc.shadowBlur = 30;
				xc.shadowColor = `rgba(255,40,0,${glow * I * 0.6})`;
				xc.stroke();

				xc.beginPath();
				cr.pts.forEach((p, i) => {
					const px = p.x * w + Math.sin(t * 4 + i) * 4;
					const py = p.y * h;
					if (i === 0) xc.moveTo(px, py);
					else xc.lineTo(px, py);
				});
				xc.strokeStyle = `rgba(255,200,50,${glow * I * 0.8})`;
				xc.lineWidth = cr.width * 0.6;
				xc.stroke();
				xc.shadowBlur = 0;

				xc.beginPath();
				cr.pts.forEach((p, i) => {
					const px = p.x * w + Math.sin(t * 4 + i) * 4;
					const py = p.y * h;
					if (i === 0) xc.moveTo(px, py);
					else xc.lineTo(px, py);
				});
				xc.strokeStyle = `rgba(255,255,200,${glow * I * 0.4})`;
				xc.lineWidth = cr.width * 0.2;
				xc.stroke();
			});

			const magmaGrd = xc.createLinearGradient(0, h * 0.7, 0, h);
			magmaGrd.addColorStop(0, 'transparent');
			magmaGrd.addColorStop(0.5, `rgba(255,40,0,${I * 0.15})`);
			magmaGrd.addColorStop(0.8, `rgba(255,80,0,${I * 0.25})`);
			magmaGrd.addColorStop(1, `rgba(255,120,20,${I * 0.3})`);
			xc.fillStyle = magmaGrd;
			xc.fillRect(0, h * 0.7, w, h * 0.3);

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s2',
			start: 'top 60%',
			onEnter: () => {
				on = true;
				const tl = gsap.timeline();
				tl.to(intensity, { v: 1, duration: 4, ease: 'power2.in' }, 0);
				tl.to('#s2f', { opacity: 0.7, duration: 0.1 }, 3.5);
				tl.to('#s2f', { opacity: 0, duration: 2 }, 3.6);
				tl.to('#s2t', { opacity: 1, duration: 0.5 }, 3.8);
				tl.from('#s2t', { scale: 3, filter: 'blur(20px)', duration: 0.5, ease: 'power4.out' }, 3.8);
				tl.add(() => shake(document.getElementById('s2-content')!, 30, 0.6), 3.8);
				tl.to('#s2s', { opacity: 1, duration: 1.2 }, 4.5);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   3. VOLTAGE — Electric God
	   ══════════════════════════════════════════════════════════════ */
	function initVoltage() {
		const cv = [s3a, s3b, s3c];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s3a.width = s3a.parentElement!.offsetWidth;
		let h = s3a.height = s3a.parentElement!.offsetHeight;
		let t = 0;
		let on = false;
		const power = { v: 0 };

		const ions: { x: number; y: number; vx: number; vy: number; size: number; charge: number }[] = [];
		for (let i = 0; i < 800; i++) {
			ions.push({
				x: Math.random() * w,
				y: Math.random() * h,
				vx: 0,
				vy: 0,
				size: Math.random() * 2.5 + 0.5,
				charge: Math.random()
			});
		}

		function bolt(ctx: CanvasRenderingContext2D, x1: number, y1: number, x2: number, y2: number, gen: number, maxGen: number, mainAlpha: number) {
			if (gen > maxGen) return;
			const segs = 6 + Math.floor(Math.random() * 6);
			const pts: { x: number; y: number }[] = [{ x: x1, y: y1 }];
			const dx = x2 - x1;
			const dy = y2 - y1;
			const len = Math.sqrt(dx * dx + dy * dy);
			const nx = -dy / len;
			const ny = dx / len;

			for (let i = 1; i < segs; i++) {
				const ti = i / segs;
				const offset = (Math.random() - 0.5) * len * 0.2 / (gen + 1);
				pts.push({ x: x1 + dx * ti + nx * offset, y: y1 + dy * ti + ny * offset });
			}
			pts.push({ x: x2, y: y2 });

			const alpha = mainAlpha / (gen * 0.7 + 1);
			ctx.beginPath();
			pts.forEach((p, i) => {
				if (i === 0) ctx.moveTo(p.x, p.y);
				else ctx.lineTo(p.x, p.y);
			});
			ctx.strokeStyle = `rgba(180,200,255,${alpha})`;
			ctx.lineWidth = Math.max(0.5, 4 / (gen + 1));
			ctx.shadowBlur = 20 / (gen + 1);
			ctx.shadowColor = `rgba(100,150,255,${alpha})`;
			ctx.stroke();

			ctx.beginPath();
			pts.forEach((p, i) => {
				if (i === 0) ctx.moveTo(p.x, p.y);
				else ctx.lineTo(p.x, p.y);
			});
			ctx.strokeStyle = `rgba(255,255,255,${alpha * 0.6})`;
			ctx.lineWidth = Math.max(0.3, 2 / (gen + 1));
			ctx.stroke();
			ctx.shadowBlur = 0;

			if (gen < maxGen - 1) {
				const branchCount = gen === 0 ? 2 : 1;
				for (let b = 0; b < branchCount; b++) {
					if (Math.random() > 0.5) continue;
					const bi = Math.floor(segs * 0.2 + Math.random() * segs * 0.6);
					const bp = pts[bi];
					if (!bp) continue;
					const ba = Math.atan2(dy, dx) + (Math.random() - 0.5) * 2;
					const bl = len * (0.2 + Math.random() * 0.3) / (gen + 1);
					bolt(ctx, bp.x, bp.y, bp.x + Math.cos(ba) * bl, bp.y + Math.sin(ba) * bl, gen + 1, maxGen, mainAlpha);
				}
			}
		}

		function resize() {
			cv.forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = s3a.width;
			h = s3a.height;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!on) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.01;
			const P = power.v;
			const [xa, xb, xc] = ctx;

			xa.fillStyle = `rgba(2,0,8,${0.2 - P * 0.05})`;
			xa.fillRect(0, 0, w, h);
			xb.clearRect(0, 0, w, h);
			xc.clearRect(0, 0, w, h);

			const cx = w / 2;
			const cy = h / 2;

			xa.globalCompositeOperation = 'lighter';
			const coreGrd = xa.createRadialGradient(cx, cy, 0, cx, cy, 100 + P * 50);
			coreGrd.addColorStop(0, `rgba(120,100,255,${P * 0.2})`);
			coreGrd.addColorStop(0.3, `rgba(80,60,200,${P * 0.1})`);
			coreGrd.addColorStop(1, 'transparent');
			xa.fillStyle = coreGrd;
			xa.fillRect(0, 0, w, h);

			const mGrd = xa.createRadialGradient(mx, my, 0, mx, my, 80);
			mGrd.addColorStop(0, `rgba(150,120,255,${P * 0.15})`);
			mGrd.addColorStop(1, 'transparent');
			xa.fillStyle = mGrd;
			xa.fillRect(mx - 80, my - 80, 160, 160);
			xa.globalCompositeOperation = 'source-over';

			xb.beginPath();
			xb.arc(cx, cy, 18 + Math.sin(t * 8) * 5, 0, Math.PI * 2);
			xb.fillStyle = `rgba(200,180,255,${P * 0.7 + Math.sin(t * 6) * 0.1})`;
			xb.shadowBlur = 60;
			xb.shadowColor = `rgba(100,80,255,${P * 0.8})`;
			xb.fill();
			xb.beginPath();
			xb.arc(cx, cy, 8 + Math.sin(t * 10) * 2, 0, Math.PI * 2);
			xb.fillStyle = `rgba(255,255,255,${P * 0.8})`;
			xb.fill();
			xb.shadowBlur = 0;

			if (P > 0.2) {
				const boltCount = Math.floor(P * 8);
				for (let i = 0; i < boltCount; i++) {
					if (i === 0 || i === 1) {
						bolt(xb, cx, cy, mx + (Math.random() - 0.5) * 30, my + (Math.random() - 0.5) * 30, 0, 3, P * 0.9);
					} else {
						const edge = Math.floor(Math.random() * 4);
						let ex, ey;
						if (edge === 0) { ex = Math.random() * w; ey = 0; }
						else if (edge === 1) { ex = w; ey = Math.random() * h; }
						else if (edge === 2) { ex = Math.random() * w; ey = h; }
						else { ex = 0; ey = Math.random() * h; }
						bolt(xb, cx, cy, ex, ey, 0, 2, P * 0.5);
					}
				}
				if (Math.random() > 0.5) {
					bolt(xb, mx, my, mx + (Math.random() - 0.5) * 300, my + (Math.random() - 0.5) * 300, 0, 2, P * 0.4);
				}
			}

			xc.globalCompositeOperation = 'lighter';
			ions.forEach(ion => {
				const dcx = cx - ion.x;
				const dcy = cy - ion.y;
				const dc = Math.sqrt(dcx * dcx + dcy * dcy);
				const dmx = mx - ion.x;
				const dmy = my - ion.y;
				const dm = Math.sqrt(dmx * dmx + dmy * dmy);
				if (dc < 300) { ion.vx += (dcx / dc) * P * 0.8; ion.vy += (dcy / dc) * P * 0.8; }
				if (dm < 200) { ion.vx += (dmx / dm) * P * 1.2; ion.vy += (dmy / dm) * P * 1.2; }
				ion.vx *= 0.94;
				ion.vy *= 0.94;
				ion.x += ion.vx;
				ion.y += ion.vy;
				if (ion.x < -50 || ion.x > w + 50 || ion.y < -50 || ion.y > h + 50) {
					ion.x = Math.random() * w;
					ion.y = Math.random() * h;
					ion.vx = 0;
					ion.vy = 0;
				}

				const speed = Math.sqrt(ion.vx * ion.vx + ion.vy * ion.vy);
				const a = Math.min(1, (0.15 + speed * 0.08) * P);
				xc.beginPath();
				xc.arc(ion.x, ion.y, ion.size * (1 + speed * 0.1), 0, Math.PI * 2);
				xc.fillStyle = `rgba(150,130,255,${a})`;
				if (speed > 3) {
					xc.shadowBlur = 8;
					xc.shadowColor = `rgba(120,100,255,${a * 0.5})`;
				}
				xc.fill();
				xc.shadowBlur = 0;

				if (speed > 2 && P > 0.5) {
					xc.beginPath();
					xc.moveTo(ion.x, ion.y);
					xc.lineTo(ion.x - ion.vx * 2, ion.y - ion.vy * 2);
					xc.strokeStyle = `rgba(150,130,255,${a * 0.4})`;
					xc.lineWidth = ion.size * 0.5;
					xc.stroke();
				}
			});
			xc.globalCompositeOperation = 'source-over';

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s3',
			start: 'top 60%',
			onEnter: () => {
				on = true;
				const tl = gsap.timeline();
				tl.to(power, { v: 1, duration: 3, ease: 'power2.in' }, 0);
				tl.to('#s3f', { opacity: 0.4, duration: 0.05 }, 2.5);
				tl.to('#s3f', { opacity: 0, duration: 0.8 }, 2.55);
				tl.add(() => shake(document.getElementById('s3-content')!, 20, 0.4), 2.5);
				tl.to('#s3t', { opacity: 1, duration: 0.4 }, 2.8);
				tl.from('#s3t', { scale: 2, filter: 'blur(15px)', duration: 0.4, ease: 'power4.out' }, 2.8);
				tl.to('#s3s', { opacity: 1, duration: 1 }, 3.5);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   4. THE MOVE — Market Nuke
	   ══════════════════════════════════════════════════════════════ */
	function initMove() {
		const cv = [s4a, s4b, s4c, s4d];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s4a.width = s4a.parentElement!.offsetWidth;
		let h = s4a.height = s4a.parentElement!.offsetHeight;
		let t = 0;
		let on = false;
		let phase = 'idle';
		let candles: { o: number; c: number; h: number; l: number; bull: boolean; visible: boolean; breakout?: boolean }[] = [];
		let shards: { x: number; y: number; vx: number; vy: number; size: number; rot: number; rv: number; life: number; color: number[] }[] = [];
		let cloudParts: { x: number; y: number; vx: number; vy: number; size: number; life: number; hue: number }[] = [];
		let shockwaves: { r: number; alpha: number; width: number }[] = [];

		let price = 5850;
		function genChart() {
			candles = [];
			for (let i = 0; i < 30; i++) {
				const o = price;
				price += (Math.random() - 0.5) * 6;
				const c = price;
				candles.push({ o, c, h: Math.max(o, c) + Math.random() * 3, l: Math.min(o, c) - Math.random() * 3, bull: c > o, visible: false });
			}
			for (let i = 0; i < 12; i++) {
				const o = price;
				price += Math.random() * 25 + 8;
				const c = price;
				candles.push({ o, c, h: c + Math.random() * 5, l: o - Math.random() * 2, bull: true, visible: false, breakout: true });
			}
		}
		genChart();

		function detonate() {
			phase = 'boom';
			const prices = candles.flatMap(c => [c.h, c.l]);
			const minP = Math.min(...prices) - 5;
			const maxP = Math.max(...prices) + 5;
			const gap = w * 0.65 / candles.length;

			for (let i = 0; i < 3000; i++) {
				const a = Math.random() * Math.PI * 2;
				const spd = 3 + Math.random() * 15;
				const isGreen = Math.random() > 0.25;
				shards.push({
					x: w * 0.17 + Math.random() * w * 0.65,
					y: h * 0.12 + Math.random() * h * 0.6,
					vx: Math.cos(a) * spd,
					vy: Math.sin(a) * spd,
					size: Math.random() * 6 + 2,
					rot: 0,
					rv: (Math.random() - 0.5) * 0.3,
					life: 1,
					color: isGreen ? [0, 255, 136] : [255, 220, 50]
				});
			}

			for (let i = 0; i < 800; i++) {
				const a = Math.random() * Math.PI * 2;
				const spd = Math.random() * 6 + 1;
				cloudParts.push({
					x: w / 2,
					y: h * 0.4,
					vx: Math.cos(a) * spd,
					vy: Math.sin(a) * spd * 0.6 - Math.random() * 4,
					size: Math.random() * 20 + 8,
					life: 1,
					hue: 100 + Math.random() * 40
				});
			}

			shockwaves.push({ r: 0, alpha: 0.8, width: 6 });
			timeouts.push(window.setTimeout(() => shockwaves.push({ r: 0, alpha: 0.6, width: 4 }), 100));
		}

		function resize() {
			cv.forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = s4a.width;
			h = s4a.height;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!on) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.008;
			const [xa, xb, xc, xd] = ctx;

			xa.fillStyle = `rgba(0,0,0,${phase === 'boom' ? 0.06 : 0.1})`;
			xa.fillRect(0, 0, w, h);
			xb.clearRect(0, 0, w, h);
			xc.clearRect(0, 0, w, h);
			xd.clearRect(0, 0, w, h);

			if (phase === 'build') {
				const vis = candles.filter(c => c.visible);
				if (vis.length === 0) {
					rafIds.push(requestAnimationFrame(draw));
					return;
				}
				const prices = vis.flatMap(c => [c.h, c.l]);
				const minP = Math.min(...prices) - 5;
				const maxP = Math.max(...prices) + 5;
				const range = maxP - minP;
				const gap = w * 0.65 / candles.length;
				const cw = Math.min(gap * 0.65, 11);
				const offX = w * 0.17;

				vis.forEach((c, idx) => {
					const i = candles.indexOf(c);
					const x = offX + i * gap + gap / 2;
					const oY = h * 0.12 + (maxP - c.o) / range * h * 0.6;
					const cY = h * 0.12 + (maxP - c.c) / range * h * 0.6;
					const hY = h * 0.12 + (maxP - c.h) / range * h * 0.6;
					const lY = h * 0.12 + (maxP - c.l) / range * h * 0.6;
					const green = c.bull;
					const color = green ? '0,255,136' : '255,51,68';

					xb.strokeStyle = `rgba(${color},0.6)`;
					xb.lineWidth = 1.5;
					xb.beginPath();
					xb.moveTo(x, hY);
					xb.lineTo(x, lY);
					xb.stroke();

					const top = Math.min(oY, cY);
					const bh = Math.abs(cY - oY) || 1.5;
					xb.fillStyle = `rgba(${color},0.85)`;
					xb.fillRect(x - cw / 2, top, cw, bh);

					if (c.breakout) {
						xb.shadowBlur = 25;
						xb.shadowColor = 'rgba(0,255,136,0.7)';
						xb.fillRect(x - cw / 2, top, cw, bh);
						xb.shadowBlur = 0;
					}
				});

				const lastVis = vis[vis.length - 1];
				if (lastVis) {
					xb.fillStyle = 'rgba(0,255,136,0.8)';
					xb.font = 'bold 14px "JetBrains Mono"';
					const ly = h * 0.12 + (maxP - lastVis.c) / (maxP - minP) * h * 0.6;
					xb.fillText(`$${lastVis.c.toFixed(2)}`, w * 0.85, ly + 4);
				}
			}

			if (phase === 'boom') {
				xa.globalCompositeOperation = 'lighter';
				shards.forEach(s => {
					s.vy += 0.06;
					s.x += s.vx;
					s.y += s.vy;
					s.vx *= 0.997;
					s.rot += s.rv;
					s.life -= 0.004;
					if (s.life > 0) {
						xa.save();
						xa.translate(s.x, s.y);
						xa.rotate(s.rot);
						xa.fillStyle = `rgba(${s.color.join(',')},${s.life * 0.9})`;
						xa.shadowBlur = 6;
						xa.shadowColor = `rgba(${s.color.join(',')},${s.life * 0.4})`;
						xa.fillRect(-s.size / 2, -s.size / 3, s.size, s.size * 0.6);
						xa.shadowBlur = 0;
						xa.restore();
					}
				});
				shards = shards.filter(s => s.life > 0);

				cloudParts.forEach(p => {
					p.vy -= 0.02;
					p.vx *= 0.995;
					p.x += p.vx;
					p.y += p.vy;
					p.life -= 0.003;
					p.size *= 1.002;
					if (p.life > 0) {
						xa.beginPath();
						xa.arc(p.x, p.y, p.size, 0, Math.PI * 2);
						xa.fillStyle = `hsla(${p.hue},80%,50%,${p.life * 0.15})`;
						xa.fill();
					}
				});
				cloudParts = cloudParts.filter(p => p.life > 0);
				xa.globalCompositeOperation = 'source-over';

				shockwaves.forEach(sw => {
					sw.r += 12;
					sw.alpha *= 0.97;
					xc.beginPath();
					xc.arc(w / 2, h * 0.4, sw.r, 0, Math.PI * 2);
					xc.strokeStyle = `rgba(255,255,255,${sw.alpha})`;
					xc.lineWidth = sw.width;
					xc.stroke();
				});
				shockwaves = shockwaves.filter(sw => sw.alpha > 0.01);
			}

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s4',
			start: 'top 60%',
			onEnter: () => {
				on = true;
				phase = 'build';
				const tl = gsap.timeline();
				candles.forEach((c, i) => {
					const delay = i < 30 ? i * 0.07 : 30 * 0.07 + (i - 30) * 0.04;
					tl.add(() => { c.visible = true; }, delay);
				});
				const boomTime = 30 * 0.07 + 12 * 0.04 + 0.3;
				tl.add(() => detonate(), boomTime);
				tl.to('#s4f', { opacity: 1, duration: 0.04 }, boomTime);
				tl.to('#s4f', { opacity: 0, duration: 2, ease: 'power3.out' }, boomTime + 0.04);
				tl.add(() => shake(document.getElementById('s4-content')!, 60, 1), boomTime);
				tl.to('#s4t', { opacity: 1, duration: 0.6 }, boomTime + 0.5);
				tl.from('#s4t', { scale: 5, filter: 'blur(40px)', duration: 0.6, ease: 'power4.out' }, boomTime + 0.5);
				tl.to('#s4s', { opacity: 1, duration: 1.2 }, boomTime + 1.5);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   5. RTP MATRIX — Grand finale
	   ══════════════════════════════════════════════════════════════ */
	function initMatrix() {
		const cv = [s5a, s5b, s5c, s5d];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s5a.width = s5a.parentElement!.offsetWidth;
		let h = s5a.height = s5a.parentElement!.offsetHeight;
		let t = 0;
		let on = false;

		const CHARS = 'アイウエオカキクケコサシスセソタチツテトナニヌネノ0123456789ABCDEF$€¥∞Γ';
		const charArr = CHARS.split('');
		const fontSize = 16;
		let cols: number;
		let drops: number[] = [];
		let colSpeed: number[] = [];
		let colBright: number[] = [];

		function initMatrix() {
			cols = Math.ceil((w || 1920) / fontSize);
			drops = [];
			colSpeed = [];
			colBright = [];
			for (let i = 0; i < cols; i++) {
				drops[i] = Math.random() * -40;
				colSpeed[i] = 0.3 + Math.random() * 0.8;
				colBright[i] = 0.4 + Math.random() * 0.6;
			}
		}

		const chartCandles: { o: number; c: number; h: number; l: number; bull: boolean }[] = [];
		let chartPrice = 5890;
		function addCandle() {
			const o = chartPrice;
			const vol = 3 + Math.random() * 12;
			chartPrice += (Math.random() - 0.47) * vol + Math.sin(chartCandles.length * 0.08) * 2;
			const c = chartPrice;
			chartCandles.push({ o, c, h: Math.max(o, c) + Math.random() * vol * 0.3, l: Math.min(o, c) - Math.random() * vol * 0.3, bull: c >= o });
			if (chartCandles.length > 50) chartCandles.shift();
		}
		for (let i = 0; i < 20; i++) addCandle();

		const parts: { x: number; y: number; vx: number; vy: number; size: number; pulse: number }[] = [];
		for (let i = 0; i < 300; i++) {
			parts.push({
				x: Math.random() * w,
				y: Math.random() * h,
				vx: (Math.random() - 0.5) * 0.5,
				vy: (Math.random() - 0.5) * 0.5,
				size: Math.random() * 2 + 0.5,
				pulse: Math.random() * 6.28
			});
		}

		function resize() {
			cv.forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = s5a.width;
			h = s5a.height;
			initMatrix();
		}
		window.addEventListener('resize', resize);
		initMatrix();

		function draw() {
			if (!on) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.008;
			const [xa, xb, xc, xd] = ctx;

			xa.fillStyle = 'rgba(0,0,0,0.07)';
			xa.fillRect(0, 0, w, h);
			xa.font = `${fontSize}px "JetBrains Mono"`;

			for (let i = 0; i < cols; i++) {
				const x = i * fontSize;
				const row = Math.floor(drops[i]);
				const y = row * fontSize;

				const dist = Math.hypot(x - mx, y - my);
				const boost = Math.max(0, 1 - dist / 200) * 0.5;
				const bright = colBright[i] + boost;

				if (y > 0 && y < h) {
					const ch = charArr[Math.floor(Math.random() * charArr.length)];
					xa.fillStyle = `rgba(200,255,230,${Math.min(1, bright)})`;
					xa.shadowBlur = 12;
					xa.shadowColor = `rgba(0,255,136,${bright * 0.8})`;
					xa.fillText(ch, x, y);
					xa.shadowBlur = 0;

					for (let tr = 1; tr < 20; tr++) {
						const ty = y - tr * fontSize;
						if (ty < 0) break;
						const ta = (1 - tr / 20) * bright * 0.5;
						if (ta < 0.02) continue;
						const tc = charArr[(row - tr + 100 * charArr.length) % charArr.length];
						const g = Math.floor(255 * (1 - tr / 20));
						xa.fillStyle = `rgba(0,${g},${Math.floor(g * 0.5)},${ta})`;
						xa.fillText(tc, x, ty);
					}
				}

				drops[i] += colSpeed[i];
				if (drops[i] * fontSize > h && Math.random() > 0.97) {
					drops[i] = Math.random() * -10;
					colSpeed[i] = 0.3 + Math.random() * 0.8;
				}
			}

			xb.clearRect(0, 0, w, h);
			if (chartCandles.length > 2) {
				const ps = chartCandles.flatMap(c => [c.h, c.l]);
				const minP = Math.min(...ps) - 5;
				const maxP = Math.max(...ps) + 5;
				const range = maxP - minP;
				const chartL = w * 0.2;
				const chartR = w * 0.8;
				const chartT = h * 0.25;
				const chartB = h * 0.7;
				const cW = Math.min((chartR - chartL) / chartCandles.length * 0.65, 10);
				const gap = (chartR - chartL) / chartCandles.length;

				chartCandles.forEach((c, i) => {
					const x = chartL + i * gap + gap / 2;
					const oY = chartT + (maxP - c.o) / range * (chartB - chartT);
					const cY = chartT + (maxP - c.c) / range * (chartB - chartT);
					const hY = chartT + (maxP - c.h) / range * (chartB - chartT);
					const lY = chartT + (maxP - c.l) / range * (chartB - chartT);
					const color = c.bull ? '0,255,136' : '255,51,68';

					xb.strokeStyle = `rgba(${color},0.5)`;
					xb.lineWidth = 1;
					xb.beginPath();
					xb.moveTo(x, hY);
					xb.lineTo(x, lY);
					xb.stroke();
					xb.fillStyle = `rgba(${color},0.7)`;
					xb.fillRect(x - cW / 2, Math.min(oY, cY), cW, Math.abs(cY - oY) || 1);

					if (i === chartCandles.length - 1) {
						xb.shadowBlur = 20;
						xb.shadowColor = `rgba(${color},0.6)`;
						xb.fillRect(x - cW / 2, Math.min(oY, cY), cW, Math.abs(cY - oY) || 1);
						xb.shadowBlur = 0;

						xb.setLineDash([4, 4]);
						xb.strokeStyle = `rgba(${color},0.15)`;
						xb.beginPath();
						xb.moveTo(chartL, cY);
						xb.lineTo(chartR + 50, cY);
						xb.stroke();
						xb.setLineDash([]);
						xb.fillStyle = `rgba(${color},0.6)`;
						xb.font = '11px "JetBrains Mono"';
						xb.fillText(`$${c.c.toFixed(2)}`, chartR + 10, cY + 4);
					}
				});
			}

			xc.clearRect(0, 0, w, h);
			parts.forEach(p => {
				p.pulse += 0.03;
				const dx = mx - p.x;
				const dy = my - p.y;
				const dist = Math.sqrt(dx * dx + dy * dy);
				if (dist < 180) { p.vx += (dx / dist) * 0.3; p.vy += (dy / dist) * 0.3; }
				p.vx *= 0.97;
				p.vy *= 0.97;
				p.x += p.vx;
				p.y += p.vy;
				if (p.x < -30) p.x = w + 30;
				if (p.x > w + 30) p.x = -30;
				if (p.y < -30) p.y = h + 30;
				if (p.y > h + 30) p.y = -30;

				const a = 0.2 + Math.sin(p.pulse) * 0.15;
				xc.beginPath();
				xc.arc(p.x, p.y, p.size, 0, Math.PI * 2);
				xc.fillStyle = `rgba(0,255,136,${a})`;
				xc.fill();
			});

			for (let i = 0; i < parts.length; i++) {
				for (let j = i + 1; j < parts.length; j++) {
					const dx = parts[i].x - parts[j].x;
					const dy = parts[i].y - parts[j].y;
					if (dx * dx + dy * dy < 6000) {
						xc.beginPath();
						xc.moveTo(parts[i].x, parts[i].y);
						xc.lineTo(parts[j].x, parts[j].y);
						xc.strokeStyle = 'rgba(0,255,136,0.06)';
						xc.lineWidth = 0.5;
						xc.stroke();
					}
				}
			}

			xd.clearRect(0, 0, w, h);
			const bsz = 50;
			const pad = 25;
			xd.strokeStyle = 'rgba(0,255,136,0.3)';
			xd.lineWidth = 1.5;
			xd.beginPath();
			xd.moveTo(pad, pad + bsz);
			xd.lineTo(pad, pad);
			xd.lineTo(pad + bsz, pad);
			xd.stroke();

			xd.beginPath();
			xd.moveTo(w - pad - bsz, pad);
			xd.lineTo(w - pad, pad);
			xd.lineTo(w - pad, pad + bsz);
			xd.stroke();

			xd.beginPath();
			xd.moveTo(pad, h - pad - bsz);
			xd.lineTo(pad, h - pad);
			xd.lineTo(pad + bsz, h - pad);
			xd.stroke();

			xd.beginPath();
			xd.moveTo(w - pad - bsz, h - pad);
			xd.lineTo(w - pad, h - pad);
			xd.lineTo(w - pad, h - pad - bsz);
			xd.stroke();

			xd.fillStyle = 'rgba(0,255,136,0.4)';
			xd.font = '10px "Orbitron"';
			xd.fillText('◈ RTP TERMINAL v4.0', pad + 10, pad + 18);
			const now = new Date();
			xd.textAlign = 'right';
			xd.fillText(`${now.toTimeString().split(' ')[0]} // LIVE`, w - pad - 10, pad + 18);
			xd.textAlign = 'start';

			xd.save();
			xd.translate(w / 2, h / 2);
			xd.rotate(t * 0.3);
			xd.beginPath();
			xd.arc(0, 0, Math.min(w, h) * 0.35, 0, Math.PI * 2);
			xd.strokeStyle = 'rgba(0,255,136,0.06)';
			xd.lineWidth = 1;
			xd.stroke();
			for (let i = 0; i < 36; i++) {
				const a = (i / 36) * Math.PI * 2;
				const r1 = Math.min(w, h) * 0.33;
				const r2 = Math.min(w, h) * (i % 3 === 0 ? 0.31 : 0.325);
				xd.beginPath();
				xd.moveTo(Math.cos(a) * r1, Math.sin(a) * r1);
				xd.lineTo(Math.cos(a) * r2, Math.sin(a) * r2);
				xd.strokeStyle = `rgba(0,255,136,${i % 3 === 0 ? 0.12 : 0.04})`;
				xd.stroke();
			}
			xd.restore();

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s5',
			start: 'top 60%',
			onEnter: () => {
				on = true;
				chartInterval = window.setInterval(addCandle, 600);
				const tl = gsap.timeline({ delay: 0.5 });
				tl.to('#s5t', { opacity: 1, duration: 0.5 }, 1);
				tl.from('#s5t', { scale: 3, filter: 'blur(30px)', duration: 0.5, ease: 'power4.out' }, 1);
				tl.add(() => shake(document.getElementById('s5-content')!, 25, 0.5), 1);
				tl.to('#s5s', { opacity: 1, duration: 1 }, 1.8);
				tl.to('#s5s2', { opacity: 1, duration: 1 }, 2.5);
			}
		});
		triggers.push(trigger);
	}

	onMount(() => {
		mx = window.innerWidth / 2;
		my = window.innerHeight / 2;
		prevMx = mx;
		prevMy = my;

		rafIds.push(requestAnimationFrame(updateMouseVelocity));

		initSingularity();
		initInferno();
		initVoltage();
		initMove();
		initMatrix();
	});

	onDestroy(() => {
		triggers.forEach(t => t.kill());
		rafIds.forEach(id => cancelAnimationFrame(id));
		timeouts.forEach(id => clearTimeout(id));
		clearInterval(chartInterval);
	});
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="s1" style="background:#020100">
	<canvas bind:this={s1a} class="z1"></canvas>
	<canvas bind:this={s1b} class="z2"></canvas>
	<canvas bind:this={s1c} class="z3"></canvas>
	<canvas bind:this={s1d} class="z4"></canvas>
	<div class="flash" id="s1f" style="background:radial-gradient(circle,#fff,transparent 40%)"></div>
	<div class="ct" id="s1-content">
		<h1 class="tt" id="s1t">SINGULARITY</h1>
		<p class="st" id="s1s">WHERE ALL RISK CONVERGES TO ZERO</p>
	</div>
	<span class="num">01</span>
</section>

<section class="hero" id="s2" style="background:#050000">
	<canvas bind:this={s2a} class="z1"></canvas>
	<canvas bind:this={s2b} class="z2"></canvas>
	<canvas bind:this={s2c} class="z3"></canvas>
	<div class="flash" id="s2f" style="background:radial-gradient(ellipse at 50% 80%,rgba(255,100,0,0.9),transparent 60%)"></div>
	<div class="ct" id="s2-content">
		<h1 class="tt" id="s2t">INFERNO</h1>
		<p class="st" id="s2s">BURN THE SHORTS. IGNITE THE MOVE.</p>
	</div>
	<span class="num">02</span>
</section>

<section class="hero" id="s3" style="background:#020008">
	<canvas bind:this={s3a} class="z1"></canvas>
	<canvas bind:this={s3b} class="z2"></canvas>
	<canvas bind:this={s3c} class="z3"></canvas>
	<div class="flash" id="s3f" style="background:rgba(100,150,255,0.7)"></div>
	<div class="ct" id="s3-content">
		<h1 class="tt" id="s3t">VOLTAGE</h1>
		<p class="st" id="s3s">THE CHARGE BEFORE THE BREAKOUT</p>
	</div>
	<span class="num">03</span>
</section>

<section class="hero" id="s4" style="background:#000502">
	<canvas bind:this={s4a} class="z1"></canvas>
	<canvas bind:this={s4b} class="z2"></canvas>
	<canvas bind:this={s4c} class="z3"></canvas>
	<canvas bind:this={s4d} class="z4"></canvas>
	<div class="flash" id="s4f" style="background:#fff"></div>
	<div class="ct" id="s4-content">
		<h1 class="tt" id="s4t">THE MOVE PRIOR<br>TO THE MOVE</h1>
		<p class="st" id="s4s">EXPLOSIVE SWINGS — REVOLUTION TRADING PROS</p>
	</div>
	<span class="num">04</span>
</section>

<section class="hero" id="s5" style="background:#000502">
	<canvas bind:this={s5a} class="z1"></canvas>
	<canvas bind:this={s5b} class="z2"></canvas>
	<canvas bind:this={s5c} class="z3"></canvas>
	<canvas bind:this={s5d} class="z4"></canvas>
	<div class="ct" id="s5-content">
		<h1 class="tt" id="s5t">RTP</h1>
		<p class="st" id="s5s">REVOLUTION TRADING PROS</p>
		<p class="st2" id="s5s2">[ 18,000+ TRADERS // THE MOVE PRIOR TO THE MOVE ]</p>
	</div>
	<span class="num">05</span>
</section>

<div style="height:10vh;background:#000"></div>

<style>
	:global(*) { margin: 0; padding: 0; box-sizing: border-box; }
	:global(html) { overflow-x: hidden; background: #000; }
	:global(body) { color: #fff; font-family: 'JetBrains Mono', monospace; }

	.hero {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		display: flex;
		align-items: center;
		justify-content: center;
		background: #000;
		color: #fff;
		font-family: 'JetBrains Mono', monospace;
	}

	.hero canvas {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
	}

	.z1 { z-index: 1; }
	.z2 { z-index: 2; }
	.z3 { z-index: 3; }
	.z4 { z-index: 4; }

	.ct {
		position: relative;
		z-index: 20;
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 100%;
		pointer-events: none;
	}

	.flash {
		position: absolute;
		inset: 0;
		z-index: 25;
		pointer-events: none;
		opacity: 0;
	}

	.tt {
		font-family: 'Anton', sans-serif;
		font-size: clamp(5rem, 18vw, 16rem);
		line-height: 0.8;
		opacity: 0;
		letter-spacing: -0.02em;
	}

	.st {
		font-family: 'Orbitron', sans-serif;
		font-size: clamp(0.5rem, 1.2vw, 0.75rem);
		letter-spacing: 0.8em;
		opacity: 0;
		margin-top: 20px;
	}

	.st2 {
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.5rem;
		letter-spacing: 0.4em;
		opacity: 0;
		margin-top: 30px;
	}

	.num {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		font-family: 'Bebas Neue';
		font-size: 60vw;
		opacity: 0.015;
		z-index: 0;
		pointer-events: none;
		color: #fff;
	}

	#s1t {
		background: linear-gradient(180deg, #fff 0%, #ff8800 30%, #ff2200 70%, #440000 100%);
		-webkit-background-clip: text;
		background-clip: text;
		color: transparent;
	}
	#s1s { color: rgba(255, 150, 50, 0.7); }

	#s2t {
		background: linear-gradient(180deg, #fff, #ffd700, #ff6600, #ff0000, #660000);
		-webkit-background-clip: text;
		background-clip: text;
		color: transparent;
	}
	#s2s { color: rgba(255, 200, 100, 0.6); }

	#s3t {
		color: #fff;
		text-shadow: 0 0 60px rgba(100, 150, 255, 0.8), 0 0 120px rgba(80, 100, 255, 0.4), 0 0 200px rgba(50, 50, 255, 0.2);
	}
	#s3s { color: rgba(150, 180, 255, 0.6); }

	#s4t {
		background: linear-gradient(180deg, #fff, #00ff88);
		-webkit-background-clip: text;
		background-clip: text;
		color: transparent;
		font-size: clamp(3rem, 10vw, 9rem);
	}
	#s4s { color: rgba(0, 255, 136, 0.6); }

	#s5t {
		color: #00ff88;
		text-shadow: 0 0 40px rgba(0, 255, 136, 0.6), 0 0 100px rgba(0, 255, 136, 0.3), 0 0 200px rgba(0, 150, 80, 0.2);
	}
	#s5s { color: rgba(0, 255, 136, 0.5); }
	#s5s2 { color: rgba(0, 255, 136, 0.3); }
</style>
