<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	// Mouse tracking
	let mx = 0;
	let my = 0;

	// Canvas refs
	let c1: HTMLCanvasElement, c2: HTMLCanvasElement, c3: HTMLCanvasElement, c4: HTMLCanvasElement;
	let c5: HTMLCanvasElement, c6: HTMLCanvasElement, c7: HTMLCanvasElement, c8: HTMLCanvasElement;
	let c9: HTMLCanvasElement, c10: HTMLCanvasElement;

	// Animation state
	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];
	let intervals: number[] = [];

	function handleMouseMove(e: MouseEvent) {
		mx = e.clientX;
		my = e.clientY;
	}

	function shake(el: HTMLElement, intensity = 20, dur = 0.5) {
		const s = Math.floor(dur / 0.025);
		for (let i = 0; i < s; i++) {
			const d = 1 - i / s;
			gsap.set(el, {
				x: (Math.random() - 0.5) * intensity * d,
				y: (Math.random() - 0.5) * intensity * d,
				delay: i * 0.025
			});
		}
		gsap.set(el, { x: 0, y: 0, delay: s * 0.025 });
	}

	/* ══════════════════════════════════════════════════════════════
	   1. SUPERNOVA — Star field + core explosion + nebula gas
	   ══════════════════════════════════════════════════════════════ */
	function initSupernova() {
		const x = c1.getContext('2d')!;
		let w = c1.width = c1.parentElement!.offsetWidth;
		let h = c1.height = c1.parentElement!.offsetHeight;
		const stars: { x: number; y: number; z: number; size: number }[] = [];
		const gas: { x: number; y: number; r: number; vx: number; vy: number; hue: number; alpha: number }[] = [];
		let t = 0;
		let active = false;
		let exploded = false;

		for (let i = 0; i < 800; i++) {
			stars.push({
				x: Math.random() * 2 - 1,
				y: Math.random() * 2 - 1,
				z: Math.random(),
				size: Math.random() * 1.5 + 0.3
			});
		}

		for (let i = 0; i < 60; i++) {
			gas.push({
				x: (Math.random() - 0.5) * w,
				y: (Math.random() - 0.5) * h,
				r: Math.random() * 150 + 50,
				vx: (Math.random() - 0.5) * 0.3,
				vy: (Math.random() - 0.5) * 0.3,
				hue: Math.random() * 60 - 10,
				alpha: 0
			});
		}

		function resize() {
			w = c1.width = c1.parentElement!.offsetWidth;
			h = c1.height = c1.parentElement!.offsetHeight;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.005;
			x.fillStyle = 'rgba(2,0,5,0.15)';
			x.fillRect(0, 0, w, h);

			const cx = w / 2;
			const cy = h / 2;
			const mouseX = (mx - cx) / w;
			const mouseY = (my - cy) / h;

			stars.forEach(s => {
				const px = cx + (s.x + mouseX * s.z * 0.3) * w * 0.6;
				const py = cy + (s.y + mouseY * s.z * 0.3) * h * 0.6;
				const flicker = 0.5 + Math.sin(t * 10 + s.x * 100) * 0.5;
				const bright = s.z * flicker;
				x.beginPath();
				x.arc(px, py, s.size * s.z, 0, Math.PI * 2);
				x.fillStyle = `rgba(255,${200 + Math.floor(s.z * 55)},${150 + Math.floor(s.z * 105)},${bright * 0.8})`;
				x.fill();
				if (s.z > 0.8) {
					x.shadowBlur = 8;
					x.shadowColor = `rgba(255,200,150,${bright * 0.3})`;
					x.fill();
					x.shadowBlur = 0;
				}
			});

			const coreR = 30 + Math.sin(t * 3) * 10;
			const grd = x.createRadialGradient(cx, cy, 0, cx, cy, coreR * 4);
			grd.addColorStop(0, `rgba(255,220,180,${0.3 + Math.sin(t * 5) * 0.1})`);
			grd.addColorStop(0.3, 'rgba(255,120,20,0.1)');
			grd.addColorStop(1, 'transparent');
			x.fillStyle = grd;
			x.fillRect(cx - coreR * 4, cy - coreR * 4, coreR * 8, coreR * 8);

			x.beginPath();
			x.arc(cx, cy, coreR * 0.3, 0, Math.PI * 2);
			x.fillStyle = `rgba(255,255,240,${0.6 + Math.sin(t * 8) * 0.2})`;
			x.shadowBlur = 40;
			x.shadowColor = 'rgba(255,180,80,0.8)';
			x.fill();
			x.shadowBlur = 0;

			x.beginPath();
			x.arc(cx, cy, 120 + Math.sin(t * 2) * 15, 0, Math.PI * 2);
			x.strokeStyle = `rgba(255,150,50,${0.05 + Math.sin(t * 3) * 0.03})`;
			x.lineWidth = 2;
			x.stroke();

			gas.forEach(g => {
				g.x += g.vx;
				g.y += g.vy;
				if (exploded && g.alpha < 0.15) g.alpha += 0.001;
				const gx = cx + g.x + mouseX * 30;
				const gy = cy + g.y + mouseY * 30;
				x.beginPath();
				x.arc(gx, gy, g.r, 0, Math.PI * 2);
				x.fillStyle = `hsla(${g.hue + t * 20},80%,50%,${g.alpha})`;
				x.fill();
			});

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s1',
			start: 'top 80%',
			onEnter: () => {
				active = true;
				const tl = gsap.timeline({ delay: 0.3 });
				tl.add(() => { exploded = true; }, 1);
				tl.fromTo('.s1-vignette',
					{ background: 'radial-gradient(ellipse,rgba(255,200,100,0.8) 0%,transparent 60%)' },
					{ background: 'radial-gradient(ellipse,transparent 30%,rgba(0,0,5,0.8) 100%)', duration: 1.5, ease: 'power3.out' }, 1);
				tl.to('#s1t', { opacity: 1, duration: 0.8, ease: 'power4.out' }, 1.5);
				tl.add(() => shake(document.getElementById('s1-content')!, 25, 0.5), 1.5);
				tl.to('#s1s', { opacity: 1, duration: 1 }, 2.2);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   2. NEURAL NETWORK — Nodes + firing synapses + electricity
	   ══════════════════════════════════════════════════════════════ */
	function initNeural() {
		const x = c2.getContext('2d')!;
		let w = c2.width = c2.parentElement!.offsetWidth;
		let h = c2.height = c2.parentElement!.offsetHeight;
		const nodes: { x: number; y: number; connections: number[]; size: number; pulse: number; baseAlpha: number }[] = [];
		const pulses: { from: number; to: number; progress: number; speed: number }[] = [];
		let t = 0;
		let active = false;

		for (let i = 0; i < 120; i++) {
			nodes.push({
				x: Math.random() * w,
				y: Math.random() * h,
				connections: [],
				size: Math.random() * 3 + 1.5,
				pulse: 0,
				baseAlpha: Math.random() * 0.3 + 0.1
			});
		}

		nodes.forEach((n, i) => {
			nodes.forEach((m, j) => {
				if (i >= j) return;
				const d = Math.hypot(n.x - m.x, n.y - m.y);
				if (d < 200) n.connections.push(j);
			});
		});

		function firePulse() {
			if (!active) return;
			const src = Math.floor(Math.random() * nodes.length);
			const n = nodes[src];
			if (n.connections.length === 0) return;
			const tgt = n.connections[Math.floor(Math.random() * n.connections.length)];
			pulses.push({ from: src, to: tgt, progress: 0, speed: 0.02 + Math.random() * 0.03 });
			nodes[src].pulse = 1;
			timeouts.push(window.setTimeout(firePulse, Math.random() * 300 + 50));
		}

		function resize() {
			w = c2.width = c2.parentElement!.offsetWidth;
			h = c2.height = c2.parentElement!.offsetHeight;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.01;
			x.fillStyle = 'rgba(0,5,8,0.08)';
			x.fillRect(0, 0, w, h);

			const mouseX = (mx / w - 0.5) * 40;
			const mouseY = (my / h - 0.5) * 40;

			nodes.forEach((n, i) => {
				n.connections.forEach(j => {
					const m = nodes[j];
					x.beginPath();
					x.moveTo(n.x + mouseX * 0.02 * (i % 3), n.y + mouseY * 0.02 * (i % 3));
					x.lineTo(m.x + mouseX * 0.02 * (j % 3), m.y + mouseY * 0.02 * (j % 3));
					x.strokeStyle = 'rgba(0,180,255,0.04)';
					x.lineWidth = 0.5;
					x.stroke();
				});
			});

			pulses.forEach(p => {
				p.progress += p.speed;
				const n = nodes[p.from];
				const m = nodes[p.to];
				const px = n.x + (m.x - n.x) * p.progress + mouseX * 0.02;
				const py = n.y + (m.y - n.y) * p.progress + mouseY * 0.02;
				x.beginPath();
				x.arc(px, py, 3, 0, Math.PI * 2);
				x.fillStyle = `rgba(0,220,255,${1 - p.progress})`;
				x.shadowBlur = 15;
				x.shadowColor = '#00ddff';
				x.fill();
				x.shadowBlur = 0;
				if (p.progress >= 1) nodes[p.to].pulse = 1;
			});
			pulses.filter(p => p.progress < 1);

			nodes.forEach((n, i) => {
				n.pulse *= 0.95;
				const md = Math.hypot(mx - n.x, my - n.y);
				const mouseGlow = Math.max(0, 1 - md / 200) * 0.4;
				const alpha = n.baseAlpha + n.pulse * 0.7 + mouseGlow;
				x.beginPath();
				x.arc(n.x + mouseX * 0.01 * (i % 5), n.y + mouseY * 0.01 * (i % 5), n.size * (1 + n.pulse * 2), 0, Math.PI * 2);
				x.fillStyle = `rgba(0,200,255,${alpha})`;
				if (n.pulse > 0.3 || mouseGlow > 0.1) {
					x.shadowBlur = 12;
					x.shadowColor = `rgba(0,220,255,${alpha})`;
				}
				x.fill();
				x.shadowBlur = 0;
			});

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s2',
			start: 'top 80%',
			onEnter: () => {
				active = true;
				firePulse();
				const tl = gsap.timeline({ delay: 0.5 });
				tl.to('#s2t', { opacity: 1, duration: 0.6, ease: 'power4.out' }, 0.5);
				tl.to('#s2s', { opacity: 1, duration: 1 }, 1.2);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   3. ABYSS — Bioluminescent particles, sonar rings, organisms
	   ══════════════════════════════════════════════════════════════ */
	function initAbyss() {
		const x = c3.getContext('2d')!;
		let w = c3.width = c3.parentElement!.offsetWidth;
		let h = c3.height = c3.parentElement!.offsetHeight;
		const orgs: { x: number; y: number; vx: number; vy: number; size: number; hue: number; pulse: number; pulseSpeed: number; trail: { x: number; y: number }[] }[] = [];
		const rings: { x: number; y: number; r: number; alpha: number }[] = [];
		let t = 0;
		let active = false;

		for (let i = 0; i < 200; i++) {
			orgs.push({
				x: Math.random() * w,
				y: Math.random() * h,
				vx: (Math.random() - 0.5) * 0.3,
				vy: -Math.random() * 0.5 - 0.1,
				size: Math.random() * 4 + 1,
				hue: 140 + Math.random() * 40,
				pulse: Math.random() * Math.PI * 2,
				pulseSpeed: 0.02 + Math.random() * 0.03,
				trail: []
			});
		}

		function sonarPing() {
			if (!active) return;
			rings.push({ x: w / 2, y: h / 2, r: 0, alpha: 0.3 });
			timeouts.push(window.setTimeout(sonarPing, 3000 + Math.random() * 2000));
		}

		function resize() {
			w = c3.width = c3.parentElement!.offsetWidth;
			h = c3.height = c3.parentElement!.offsetHeight;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.008;
			x.fillStyle = 'rgba(0,2,5,0.06)';
			x.fillRect(0, 0, w, h);

			const cx = mx / w - 0.5;
			const cy = my / h - 0.5;

			rings.forEach(r => {
				r.r += 2;
				r.alpha *= 0.98;
				x.beginPath();
				x.arc(r.x, r.y, r.r, 0, Math.PI * 2);
				x.strokeStyle = `rgba(0,255,170,${r.alpha})`;
				x.lineWidth = 1;
				x.stroke();
			});
			rings.filter(r => r.alpha > 0.01);

			orgs.forEach(o => {
				o.pulse += o.pulseSpeed;
				o.vx += cx * 0.01;
				o.vy += cy * 0.005;
				o.vx *= 0.99;
				o.vy *= 0.99;
				o.x += o.vx;
				o.y += o.vy;

				if (o.x < -20) o.x = w + 20;
				if (o.x > w + 20) o.x = -20;
				if (o.y < -20) o.y = h + 20;
				if (o.y > h + 20) o.y = -20;

				o.trail.push({ x: o.x, y: o.y });
				if (o.trail.length > 8) o.trail.shift();

				const glow = 0.3 + Math.sin(o.pulse) * 0.3;
				o.trail.forEach((tp, i) => {
					const a = (i / o.trail.length) * glow * 0.3;
					x.beginPath();
					x.arc(tp.x, tp.y, o.size * 0.5 * (i / o.trail.length), 0, Math.PI * 2);
					x.fillStyle = `hsla(${o.hue},100%,60%,${a})`;
					x.fill();
				});

				x.beginPath();
				x.arc(o.x, o.y, o.size * (0.8 + Math.sin(o.pulse) * 0.2), 0, Math.PI * 2);
				x.fillStyle = `hsla(${o.hue},100%,70%,${glow})`;
				x.shadowBlur = o.size * 4;
				x.shadowColor = `hsla(${o.hue},100%,50%,${glow * 0.5})`;
				x.fill();
				x.shadowBlur = 0;
			});

			for (let i = 0; i < 5; i++) {
				const rx = w * 0.15 + i * w * 0.18 + Math.sin(t + i) * 30;
				const grd = x.createLinearGradient(rx, 0, rx + 20, h * 0.6);
				grd.addColorStop(0, `rgba(0,100,120,${0.01 + Math.sin(t + i * 0.5) * 0.005})`);
				grd.addColorStop(1, 'transparent');
				x.fillStyle = grd;
				x.fillRect(rx - 20, 0, 60, h * 0.6);
			}

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s3',
			start: 'top 80%',
			onEnter: () => {
				active = true;
				sonarPing();
				const tl = gsap.timeline({ delay: 0.5 });
				tl.to('#s3t', { opacity: 1, duration: 1.2, ease: 'power2.out' }, 0.5);
				tl.to('#s3s', { opacity: 1, duration: 1 }, 1.5);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   4. DIGITAL STORM — Binary vortex, electromagnetic arcs
	   ══════════════════════════════════════════════════════════════ */
	function initStorm() {
		const x = c4.getContext('2d')!;
		let w = c4.width = c4.parentElement!.offsetWidth;
		let h = c4.height = c4.parentElement!.offsetHeight;
		const bits: { angle: number; dist: number; speed: number; driftOut: number; char: string; size: number; alpha: number; z: number }[] = [];
		const arcs: { points: { a: number; r: number }[]; life: number }[] = [];
		const CHARS = '01';
		let t = 0;
		let active = false;

		for (let i = 0; i < 600; i++) {
			const angle = Math.random() * Math.PI * 2;
			const dist = Math.random() * Math.min(w, h) * 0.5;
			bits.push({
				angle,
				dist,
				speed: 0.002 + Math.random() * 0.008,
				driftOut: Math.random() * 0.3 - 0.15,
				char: CHARS[Math.floor(Math.random() * 2)],
				size: Math.random() * 10 + 8,
				alpha: Math.random() * 0.6 + 0.1,
				z: Math.random()
			});
		}

		function spawnArc() {
			if (!active) return;
			const startAngle = Math.random() * Math.PI * 2;
			const points: { a: number; r: number }[] = [];
			let a = startAngle;
			let r = 50;
			for (let i = 0; i < 15; i++) {
				a += Math.random() * 0.4 - 0.2;
				r += Math.random() * 30 + 10;
				points.push({ a, r });
			}
			arcs.push({ points, life: 1 });
			timeouts.push(window.setTimeout(spawnArc, Math.random() * 500 + 100));
		}

		function resize() {
			w = c4.width = c4.parentElement!.offsetWidth;
			h = c4.height = c4.parentElement!.offsetHeight;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.01;
			x.fillStyle = 'rgba(3,3,3,0.12)';
			x.fillRect(0, 0, w, h);

			const cx = w / 2 + (mx / w - 0.5) * 30;
			const cy = h / 2 + (my / h - 0.5) * 30;

			x.font = '10px "JetBrains Mono"';
			bits.forEach(b => {
				b.angle += b.speed * (1 + b.z);
				b.dist += b.driftOut;
				if (b.dist > Math.max(w, h) * 0.7) b.dist = 20 + Math.random() * 50;
				if (b.dist < 10) b.dist = Math.min(w, h) * 0.5;

				if (Math.random() > 0.99) b.char = CHARS[Math.floor(Math.random() * 2)];

				const px = cx + Math.cos(b.angle) * b.dist;
				const py = cy + Math.sin(b.angle) * b.dist;
				const distFade = Math.min(1, b.dist / 200);
				x.fillStyle = `rgba(255,255,255,${b.alpha * distFade * (0.5 + b.z * 0.5)})`;
				x.fillText(b.char, px, py);
			});

			arcs.forEach(arc => {
				arc.life *= 0.95;
				x.beginPath();
				arc.points.forEach((p, i) => {
					const jx = cx + Math.cos(p.a + t * 2) * (p.r + (Math.random() - 0.5) * 10);
					const jy = cy + Math.sin(p.a + t * 2) * (p.r + (Math.random() - 0.5) * 10);
					if (i === 0) x.moveTo(jx, jy);
					else x.lineTo(jx, jy);
				});
				x.strokeStyle = `rgba(255,255,255,${arc.life * 0.6})`;
				x.lineWidth = 1 + arc.life;
				x.shadowBlur = 15;
				x.shadowColor = `rgba(200,200,255,${arc.life * 0.5})`;
				x.stroke();
				x.shadowBlur = 0;
			});
			arcs.filter(a => a.life > 0.05);

			const eyeR = 30 + Math.sin(t * 3) * 5;
			x.beginPath();
			x.arc(cx, cy, eyeR, 0, Math.PI * 2);
			x.fillStyle = `rgba(255,255,255,${0.03 + Math.sin(t * 4) * 0.02})`;
			x.fill();
			x.strokeStyle = 'rgba(255,255,255,0.08)';
			x.lineWidth = 1;
			x.stroke();

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s4',
			start: 'top 80%',
			onEnter: () => {
				active = true;
				spawnArc();
				const tl = gsap.timeline({ delay: 0.3 });
				tl.to('#s4t', { opacity: 1, duration: 0.1, ease: 'none' }, 0.5);
				tl.add(() => shake(document.getElementById('s4-content')!, 30, 0.4), 0.5);
				tl.to('#s4s', { opacity: 1, duration: 1 }, 1.2);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   5. HEARTBEAT MARKET — EKG line, pulse-driven particles
	   ══════════════════════════════════════════════════════════════ */
	function initHeartbeat() {
		const x = c5.getContext('2d')!;
		let w = c5.width = c5.parentElement!.offsetWidth;
		let h = c5.height = c5.parentElement!.offsetHeight;
		let t = 0;
		let active = false;
		const pulseParticles: { x: number; y: number; vx: number; vy: number; life: number; size: number }[] = [];

		function ekgValue(phase: number) {
			const p = phase % 1;
			if (p < 0.1) return 0;
			if (p < 0.15) return Math.sin((p - 0.1) / 0.05 * Math.PI) * 0.3;
			if (p < 0.2) return 0;
			if (p < 0.22) return -0.15;
			if (p < 0.26) return 1;
			if (p < 0.3) return -0.3;
			if (p < 0.35) return 0;
			if (p < 0.45) return Math.sin((p - 0.35) / 0.1 * Math.PI) * 0.2;
			return 0;
		}

		function pulseBurst() {
			if (!active) return;
			for (let i = 0; i < 30; i++) {
				const angle = Math.random() * Math.PI * 2;
				const speed = Math.random() * 4 + 2;
				pulseParticles.push({
					x: w / 2,
					y: h / 2,
					vx: Math.cos(angle) * speed,
					vy: Math.sin(angle) * speed,
					life: 1,
					size: Math.random() * 3 + 1
				});
			}
			gsap.to('#s5', { boxShadow: 'inset 0 0 120px rgba(255,0,50,0.15)', duration: 0.15,
				onComplete: () => gsap.to('#s5', { boxShadow: 'inset 0 0 120px rgba(255,0,50,0)', duration: 0.8 }) });
			timeouts.push(window.setTimeout(pulseBurst, 800 + Math.random() * 200));
		}

		function resize() {
			w = c5.width = c5.parentElement!.offsetWidth;
			h = c5.height = c5.parentElement!.offsetHeight;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.012;
			x.fillStyle = 'rgba(5,0,0,0.08)';
			x.fillRect(0, 0, w, h);

			x.beginPath();
			const ekgY = h * 0.5;
			for (let i = 0; i < w; i += 2) {
				const phase = ((i / w) * 2 + t) % 2;
				const val = ekgValue(phase % 1) * 150;
				const px = i;
				const py = ekgY - val;
				if (i === 0) x.moveTo(px, py);
				else x.lineTo(px, py);
			}
			x.strokeStyle = 'rgba(255,0,50,0.4)';
			x.lineWidth = 2;
			x.shadowBlur = 20;
			x.shadowColor = 'rgba(255,0,50,0.5)';
			x.stroke();
			x.shadowBlur = 0;

			for (const offset of [-50, 50, -100, 100]) {
				x.beginPath();
				for (let i = 0; i < w; i += 3) {
					const phase = ((i / w) * 2 + t + offset * 0.001) % 2;
					const val = ekgValue(phase % 1) * 100;
					if (i === 0) x.moveTo(i, ekgY + offset - val);
					else x.lineTo(i, ekgY + offset - val);
				}
				x.strokeStyle = 'rgba(255,0,50,0.06)';
				x.lineWidth = 1;
				x.stroke();
			}

			const pulsePhase = (t * 1.2) % 1;
			const pulseR = pulsePhase * Math.max(w, h) * 0.5;
			x.beginPath();
			x.arc(w / 2, h / 2, pulseR, 0, Math.PI * 2);
			x.strokeStyle = `rgba(255,0,50,${(1 - pulsePhase) * 0.1})`;
			x.lineWidth = 2;
			x.stroke();

			pulseParticles.forEach(p => {
				p.x += p.vx;
				p.y += p.vy;
				p.vx *= 0.98;
				p.vy *= 0.98;
				p.life -= 0.015;
				x.beginPath();
				x.arc(p.x, p.y, p.size * p.life, 0, Math.PI * 2);
				x.fillStyle = `rgba(255,30,60,${p.life * 0.6})`;
				x.shadowBlur = 8;
				x.shadowColor = 'rgba(255,0,50,0.4)';
				x.fill();
				x.shadowBlur = 0;
			});
			pulseParticles.filter(p => p.life > 0);

			const bpm = Math.floor(72 + Math.sin(t * 0.5) * 8);
			x.fillStyle = 'rgba(255,0,50,0.15)';
			x.font = 'bold 80px Orbitron';
			x.textAlign = 'right';
			x.fillText(bpm.toString(), w - 40, h - 40);
			x.font = '12px "JetBrains Mono"';
			x.fillText('BPM', w - 40, h - 15);
			x.textAlign = 'start';

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s5',
			start: 'top 80%',
			onEnter: () => {
				active = true;
				pulseBurst();
				const tl = gsap.timeline({ delay: 0.3 });
				tl.to('#s5t', { opacity: 1, duration: 0.15, ease: 'power4.out' }, 0.8);
				tl.add(() => shake(document.getElementById('s5-content')!, 20, 0.3), 0.8);
				tl.to('#s5s', { opacity: 1, duration: 1 }, 1.5);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   6. BLUEPRINT — Technical grid, measurement annotations
	   ══════════════════════════════════════════════════════════════ */
	function initBlueprint() {
		const x = c6.getContext('2d')!;
		let w = c6.width = c6.parentElement!.offsetWidth;
		let h = c6.height = c6.parentElement!.offsetHeight;
		let t = 0;
		let active = false;
		const shapes: { cx: number; cy: number; type: number; size: number; progress: number; rotation: number }[] = [];

		for (let i = 0; i < 12; i++) {
			const cx = Math.random() * w * 0.6 + w * 0.2;
			const cy = Math.random() * h * 0.6 + h * 0.2;
			const type = Math.floor(Math.random() * 3);
			shapes.push({ cx, cy, type, size: Math.random() * 80 + 40, progress: 0, rotation: Math.random() * Math.PI * 2 });
		}

		function resize() {
			w = c6.width = c6.parentElement!.offsetWidth;
			h = c6.height = c6.parentElement!.offsetHeight;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.008;
			x.clearRect(0, 0, w, h);

			x.strokeStyle = 'rgba(68,153,255,0.04)';
			x.lineWidth = 0.5;
			const gridS = 40;
			for (let gx = 0; gx < w; gx += gridS) {
				x.beginPath();
				x.moveTo(gx, 0);
				x.lineTo(gx, h);
				x.stroke();
			}
			for (let gy = 0; gy < h; gy += gridS) {
				x.beginPath();
				x.moveTo(0, gy);
				x.lineTo(w, gy);
				x.stroke();
			}

			x.strokeStyle = 'rgba(68,153,255,0.08)';
			x.lineWidth = 1;
			for (let gx = 0; gx < w; gx += gridS * 5) {
				x.beginPath();
				x.moveTo(gx, 0);
				x.lineTo(gx, h);
				x.stroke();
			}
			for (let gy = 0; gy < h; gy += gridS * 5) {
				x.beginPath();
				x.moveTo(0, gy);
				x.lineTo(w, gy);
				x.stroke();
			}

			x.strokeStyle = 'rgba(68,153,255,0.2)';
			x.setLineDash([4, 4]);
			x.lineWidth = 0.5;
			x.beginPath();
			x.moveTo(mx, 0);
			x.lineTo(mx, h);
			x.stroke();
			x.beginPath();
			x.moveTo(0, my);
			x.lineTo(w, my);
			x.stroke();
			x.setLineDash([]);

			x.fillStyle = 'rgba(68,153,255,0.25)';
			x.font = '9px "JetBrains Mono"';
			x.fillText(`x:${mx.toFixed(0)}`, mx + 8, my - 8);
			x.fillText(`y:${my.toFixed(0)}`, mx + 8, my + 15);

			shapes.forEach(s => {
				if (s.progress <= 0) return;
				const p = Math.min(s.progress, 1);
				x.save();
				x.translate(s.cx, s.cy);
				x.rotate(s.rotation + t * 0.1);

				x.strokeStyle = `rgba(68,153,255,${0.3 * p})`;
				x.lineWidth = 1;

				if (s.type === 0) {
					const hw = (s.size / 2) * p;
					const hh = (s.size * 0.6 / 2) * p;
					x.strokeRect(-hw, -hh, hw * 2, hh * 2);
					x.strokeStyle = 'rgba(68,153,255,0.1)';
					x.setLineDash([2, 2]);
					x.beginPath();
					x.moveTo(-hw, -hh - 15);
					x.lineTo(hw, -hh - 15);
					x.stroke();
					x.setLineDash([]);
					x.fillStyle = 'rgba(68,153,255,0.15)';
					x.font = '7px "JetBrains Mono"';
					x.textAlign = 'center';
					x.fillText((s.size * p).toFixed(0) + 'px', 0, -hh - 20);
					x.textAlign = 'start';
				} else if (s.type === 1) {
					x.beginPath();
					x.arc(0, 0, (s.size / 2) * p, 0, Math.PI * 2 * p);
					x.stroke();
					x.strokeStyle = 'rgba(68,153,255,0.1)';
					x.setLineDash([2, 2]);
					x.beginPath();
					x.moveTo(0, 0);
					x.lineTo((s.size / 2) * p, 0);
					x.stroke();
					x.setLineDash([]);
				} else {
					const r = (s.size / 2) * p;
					x.beginPath();
					x.moveTo(0, -r);
					x.lineTo(r * 0.87, r * 0.5);
					x.lineTo(-r * 0.87, r * 0.5);
					x.closePath();
					x.stroke();
				}
				x.fillStyle = `rgba(68,153,255,${0.5 * p})`;
				x.beginPath();
				x.arc(0, 0, 2, 0, Math.PI * 2);
				x.fill();

				x.restore();
			});

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s6',
			start: 'top 80%',
			onEnter: () => {
				active = true;
				shapes.forEach((s, i) => {
					gsap.to(s, { progress: 1, duration: 1.5, delay: i * 0.15, ease: 'power2.out' });
				});
				const tl = gsap.timeline({ delay: 1 });
				tl.to('#s6t', { opacity: 1, duration: 0.8 }, 0.5);
				tl.to('#s6s', { opacity: 1, duration: 1 }, 1.2);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   7. PREDATOR — Thermal signatures, lock-on brackets, scanning
	   ══════════════════════════════════════════════════════════════ */
	function initPredator() {
		const x = c7.getContext('2d')!;
		let w = c7.width = c7.parentElement!.offsetWidth;
		let h = c7.height = c7.parentElement!.offsetHeight;
		let t = 0;
		let active = false;
		const targets: { x: number; y: number; size: number; locked: boolean; heat: number }[] = [];

		for (let i = 0; i < 8; i++) {
			targets.push({
				x: Math.random() * w * 0.6 + w * 0.2,
				y: Math.random() * h * 0.6 + h * 0.2,
				size: Math.random() * 40 + 20,
				locked: false,
				heat: Math.random() * 0.5 + 0.5
			});
		}

		function resize() {
			w = c7.width = c7.parentElement!.offsetWidth;
			h = c7.height = c7.parentElement!.offsetHeight;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.01;
			x.fillStyle = 'rgba(2,2,0,0.1)';
			x.fillRect(0, 0, w, h);

			const scanAngle = t * 0.8;
			const scanX = w / 2 + Math.cos(scanAngle) * w * 0.4;
			const scanGrd = x.createLinearGradient(w / 2, h / 2, scanX, h / 2 + Math.sin(scanAngle) * h * 0.3);
			scanGrd.addColorStop(0, 'transparent');
			scanGrd.addColorStop(1, 'rgba(255,200,0,0.02)');
			x.fillStyle = scanGrd;
			x.beginPath();
			x.moveTo(w / 2, h / 2);
			x.arc(w / 2, h / 2, w * 0.5, scanAngle - 0.3, scanAngle + 0.3);
			x.closePath();
			x.fill();

			for (let i = 0; i < 50; i++) {
				const nx = Math.random() * w;
				const ny = Math.random() * h;
				x.fillStyle = `rgba(${Math.random() * 50 + 20},${Math.random() * 30 + 10},0,0.02)`;
				x.fillRect(nx, ny, Math.random() * 20 + 5, Math.random() * 20 + 5);
			}

			targets.forEach(tgt => {
				const grd = x.createRadialGradient(tgt.x, tgt.y, 0, tgt.x, tgt.y, tgt.size);
				grd.addColorStop(0, `rgba(255,200,0,${tgt.heat * 0.15 + Math.sin(t * 3) * 0.03})`);
				grd.addColorStop(0.5, `rgba(255,100,0,${tgt.heat * 0.08})`);
				grd.addColorStop(1, 'transparent');
				x.fillStyle = grd;
				x.fillRect(tgt.x - tgt.size, tgt.y - tgt.size, tgt.size * 2, tgt.size * 2);
			});

			const bs = 40 + Math.sin(t * 5) * 5;
			x.strokeStyle = `rgba(255,200,0,${0.5 + Math.sin(t * 8) * 0.2})`;
			x.lineWidth = 2;
			x.beginPath();
			x.moveTo(mx - bs, my - bs + 12);
			x.lineTo(mx - bs, my - bs);
			x.lineTo(mx - bs + 12, my - bs);
			x.stroke();

			x.beginPath();
			x.moveTo(mx + bs - 12, my - bs);
			x.lineTo(mx + bs, my - bs);
			x.lineTo(mx + bs, my - bs + 12);
			x.stroke();

			x.beginPath();
			x.moveTo(mx + bs, my + bs - 12);
			x.lineTo(mx + bs, my + bs);
			x.lineTo(mx + bs - 12, my + bs);
			x.stroke();

			x.beginPath();
			x.moveTo(mx - bs + 12, my + bs);
			x.lineTo(mx - bs, my + bs);
			x.lineTo(mx - bs, my + bs - 12);
			x.stroke();

			x.beginPath();
			x.moveTo(mx, my - 8);
			x.lineTo(mx + 8, my);
			x.lineTo(mx, my + 8);
			x.lineTo(mx - 8, my);
			x.closePath();
			x.strokeStyle = `rgba(255,200,0,${0.3 + Math.sin(t * 6) * 0.2})`;
			x.lineWidth = 1;
			x.stroke();

			x.fillStyle = 'rgba(255,200,0,0.3)';
			x.font = '9px "JetBrains Mono"';
			x.fillText(`DIST: ${(Math.hypot(mx - w / 2, my - h / 2) / 10).toFixed(1)}m`, mx + bs + 8, my - bs + 10);
			x.fillText(`HEAT: ${(0.5 + Math.sin(t * 2) * 0.3).toFixed(2)}`, mx + bs + 8, my - bs + 25);

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s7',
			start: 'top 80%',
			onEnter: () => {
				active = true;
				const tl = gsap.timeline({ delay: 0.3 });
				tl.to('#s7t', { opacity: 1, duration: 0.1 }, 1);
				tl.add(() => shake(document.getElementById('s7-content')!, 15, 0.3), 1);
				tl.to('#s7s', { opacity: 1, duration: 0.8 }, 1.5);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   8. TIME WARP — Speed lines converging to center, clock fragments
	   ══════════════════════════════════════════════════════════════ */
	function initTimewarp() {
		const x = c8.getContext('2d')!;
		let w = c8.width = c8.parentElement!.offsetWidth;
		let h = c8.height = c8.parentElement!.offsetHeight;
		let t = 0;
		let active = false;
		const lines: { angle: number; dist: number; speed: number; width: number; len: number; alpha: number }[] = [];
		const fragments: { text: string; x: number; y: number; vx: number; vy: number; rotation: number; rotSpeed: number; alpha: number }[] = [];

		for (let i = 0; i < 200; i++) {
			const angle = Math.random() * Math.PI * 2;
			lines.push({
				angle,
				dist: Math.random() * Math.max(w, h) * 0.7 + 100,
				speed: Math.random() * 8 + 3,
				width: Math.random() * 2 + 0.5,
				len: Math.random() * 60 + 20,
				alpha: Math.random() * 0.4 + 0.1
			});
		}

		for (let i = 1; i <= 12; i++) {
			const angle = (i / 12) * Math.PI * 2 - Math.PI / 2;
			const r = Math.min(w, h) * 0.3;
			fragments.push({
				text: String(i),
				x: Math.cos(angle) * r,
				y: Math.sin(angle) * r,
				vx: (Math.random() - 0.5) * 3,
				vy: (Math.random() - 0.5) * 3,
				rotation: 0,
				rotSpeed: (Math.random() - 0.5) * 0.05,
				alpha: 0
			});
		}

		function resize() {
			w = c8.width = c8.parentElement!.offsetWidth;
			h = c8.height = c8.parentElement!.offsetHeight;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.015;
			x.fillStyle = 'rgba(2,0,10,0.1)';
			x.fillRect(0, 0, w, h);

			const cx = w / 2 + (mx / w - 0.5) * 40;
			const cy = h / 2 + (my / h - 0.5) * 40;

			lines.forEach(l => {
				l.dist -= l.speed;
				if (l.dist < 10) {
					l.dist = Math.max(w, h) * 0.8;
					l.angle = Math.random() * Math.PI * 2;
				}

				const ex = cx + Math.cos(l.angle) * l.dist;
				const ey = cy + Math.sin(l.angle) * l.dist;
				const sx = cx + Math.cos(l.angle) * (l.dist + l.len);
				const sy = cy + Math.sin(l.angle) * (l.dist + l.len);

				const fade = Math.min(1, l.dist / 200);
				x.beginPath();
				x.moveTo(sx, sy);
				x.lineTo(ex, ey);
				x.strokeStyle = `rgba(200,100,255,${l.alpha * fade})`;
				x.lineWidth = l.width;
				x.stroke();
			});

			for (let i = 0; i < 3; i++) {
				const r = 20 + i * 25 + Math.sin(t * 3 + i) * 10;
				x.beginPath();
				x.arc(cx, cy, r, t * 2 + i, t * 2 + i + Math.PI * 1.5);
				x.strokeStyle = `rgba(200,100,255,${0.1 - i * 0.025})`;
				x.lineWidth = 2;
				x.stroke();
			}

			fragments.forEach(f => {
				f.rotation += f.rotSpeed;
				const fx = cx + f.x * Math.cos(t * 0.3) - f.y * Math.sin(t * 0.3);
				const fy = cy + f.x * Math.sin(t * 0.3) + f.y * Math.cos(t * 0.3);
				x.save();
				x.translate(fx, fy);
				x.rotate(f.rotation);
				x.fillStyle = `rgba(200,100,255,${f.alpha})`;
				x.font = 'bold 20px Orbitron';
				x.textAlign = 'center';
				x.fillText(f.text, 0, 0);
				x.restore();
			});

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s8',
			start: 'top 80%',
			onEnter: () => {
				active = true;
				fragments.forEach((f, i) => {
					gsap.to(f, { alpha: 0.3, duration: 0.5, delay: i * 0.08 });
				});
				const tl = gsap.timeline({ delay: 0.5 });
				tl.to('#s8t', { opacity: 1, duration: 0.5, ease: 'power4.out' }, 0.8);
				tl.to('#s8s', { opacity: 1, duration: 1 }, 1.5);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   9. VOLCANIC — Magma cracks, rising embers, heat distortion
	   ══════════════════════════════════════════════════════════════ */
	function initVolcanic() {
		const x = c9.getContext('2d')!;
		let w = c9.width = c9.parentElement!.offsetWidth;
		let h = c9.height = c9.parentElement!.offsetHeight;
		let t = 0;
		let active = false;
		const embers: { x: number; y: number; vx: number; vy: number; size: number; life: number; hue: number }[] = [];
		const cracks: { pts: { x: number; y: number }[]; glow: number; width: number }[] = [];

		for (let i = 0; i < 150; i++) {
			embers.push({
				x: Math.random() * w,
				y: h + Math.random() * 100,
				vx: (Math.random() - 0.5) * 1,
				vy: -Math.random() * 3 - 1,
				size: Math.random() * 4 + 1,
				life: Math.random(),
				hue: Math.random() * 40 + 10
			});
		}

		for (let i = 0; i < 8; i++) {
			const pts: { x: number; y: number }[] = [];
			let cx = Math.random() * w;
			let cy = h;
			for (let j = 0; j < 10; j++) {
				cx += (Math.random() - 0.5) * 80;
				cy -= Math.random() * 60 + 20;
				pts.push({ x: cx, y: cy });
			}
			cracks.push({ pts, glow: 0, width: Math.random() * 3 + 1 });
		}

		function resize() {
			w = c9.width = c9.parentElement!.offsetWidth;
			h = c9.height = c9.parentElement!.offsetHeight;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.008;
			x.fillStyle = 'rgba(5,2,0,0.06)';
			x.fillRect(0, 0, w, h);

			const grd = x.createLinearGradient(0, h * 0.7, 0, h);
			grd.addColorStop(0, 'transparent');
			grd.addColorStop(0.5, `rgba(255,60,0,${0.03 + Math.sin(t * 2) * 0.01})`);
			grd.addColorStop(1, `rgba(255,30,0,${0.06 + Math.sin(t * 3) * 0.02})`);
			x.fillStyle = grd;
			x.fillRect(0, h * 0.7, w, h * 0.3);

			cracks.forEach(cr => {
				cr.glow = 0.3 + Math.sin(t * 2 + cr.pts[0].x * 0.01) * 0.2;
				x.beginPath();
				cr.pts.forEach((p, i) => {
					const jx = p.x + Math.sin(t * 3 + i) * 3;
					if (i === 0) x.moveTo(jx, p.y);
					else x.lineTo(jx, p.y);
				});
				x.strokeStyle = `rgba(255,${80 + Math.floor(cr.glow * 100)},0,${cr.glow})`;
				x.lineWidth = cr.width;
				x.shadowBlur = 20;
				x.shadowColor = `rgba(255,60,0,${cr.glow * 0.6})`;
				x.stroke();

				x.strokeStyle = `rgba(255,200,50,${cr.glow * 0.5})`;
				x.lineWidth = cr.width * 0.4;
				x.stroke();
				x.shadowBlur = 0;
			});

			embers.forEach(e => {
				e.x += e.vx + Math.sin(t * 2 + e.x * 0.01) * 0.5;
				e.y += e.vy;
				e.life -= 0.003;
				if (e.life <= 0 || e.y < -20) {
					e.x = Math.random() * w;
					e.y = h + 20;
					e.life = 1;
					e.vx = (Math.random() - 0.5) * 1;
					e.vy = -Math.random() * 3 - 1;
				}
				const flickr = 0.5 + Math.sin(t * 20 + e.x) * 0.5;
				x.beginPath();
				x.arc(e.x, e.y, e.size * e.life, 0, Math.PI * 2);
				x.fillStyle = `hsla(${e.hue},100%,${50 + flickr * 20}%,${e.life * 0.7 * flickr})`;
				x.shadowBlur = e.size * 3;
				x.shadowColor = `hsla(${e.hue},100%,50%,${e.life * 0.4})`;
				x.fill();
				x.shadowBlur = 0;
			});

			const mGrd = x.createRadialGradient(mx, my, 0, mx, my, 100);
			mGrd.addColorStop(0, 'rgba(255,100,0,0.04)');
			mGrd.addColorStop(1, 'transparent');
			x.fillStyle = mGrd;
			x.fillRect(mx - 100, my - 100, 200, 200);

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s9',
			start: 'top 80%',
			onEnter: () => {
				active = true;
				const tl = gsap.timeline({ delay: 0.5 });
				tl.to('#s9t', { opacity: 1, duration: 0.6 }, 0.5);
				tl.add(() => shake(document.getElementById('s9-content')!, 15, 0.4), 0.5);
				tl.to('#s9s', { opacity: 1, duration: 1 }, 1.2);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   10. CYBERPUNK — Neon rain, holographic scan lines, electric arcs
	   ══════════════════════════════════════════════════════════════ */
	function initCyberpunk() {
		const x = c10.getContext('2d')!;
		let w = c10.width = c10.parentElement!.offsetWidth;
		let h = c10.height = c10.parentElement!.offsetHeight;
		let t = 0;
		let active = false;
		const neonDrops: { x: number; y: number; speed: number; len: number; hue: number; alpha: number }[] = [];
		const arcs: { pts: { x: number; y: number }[]; life: number; hue: number }[] = [];

		const columns = Math.ceil(w / 8);
		for (let i = 0; i < columns; i++) {
			neonDrops.push({
				x: i * 8,
				y: Math.random() * -h,
				speed: Math.random() * 6 + 3,
				len: Math.random() * 100 + 50,
				hue: Math.random() > 0.5 ? 300 : 180,
				alpha: Math.random() * 0.3 + 0.1
			});
		}

		function spawnArc() {
			if (!active) return;
			const y = Math.random() * h;
			const pts: { x: number; y: number }[] = [];
			let px = 0;
			while (px < w) {
				pts.push({ x: px, y: y + (Math.random() - 0.5) * 40 });
				px += Math.random() * 30 + 10;
			}
			arcs.push({ pts, life: 1, hue: Math.random() > 0.5 ? 300 : 180 });
			timeouts.push(window.setTimeout(spawnArc, Math.random() * 2000 + 500));
		}

		function resize() {
			w = c10.width = c10.parentElement!.offsetWidth;
			h = c10.height = c10.parentElement!.offsetHeight;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.01;
			x.fillStyle = 'rgba(2,0,10,0.12)';
			x.fillRect(0, 0, w, h);

			neonDrops.forEach(d => {
				d.y += d.speed;
				if (d.y > h + d.len) {
					d.y = -d.len;
					d.x = Math.random() * w;
				}

				const grd = x.createLinearGradient(d.x, d.y - d.len, d.x, d.y);
				grd.addColorStop(0, 'transparent');
				grd.addColorStop(0.8, `hsla(${d.hue},100%,60%,${d.alpha})`);
				grd.addColorStop(1, `hsla(${d.hue},100%,80%,${d.alpha * 1.5})`);
				x.strokeStyle = grd;
				x.lineWidth = 1.5;
				x.beginPath();
				x.moveTo(d.x, d.y - d.len);
				x.lineTo(d.x, d.y);
				x.stroke();

				x.beginPath();
				x.arc(d.x, d.y, 2, 0, Math.PI * 2);
				x.fillStyle = `hsla(${d.hue},100%,80%,${d.alpha})`;
				x.shadowBlur = 10;
				x.shadowColor = `hsla(${d.hue},100%,50%,0.5)`;
				x.fill();
				x.shadowBlur = 0;
			});

			arcs.forEach(a => {
				a.life *= 0.93;
				x.beginPath();
				a.pts.forEach((p, i) => {
					const jy = p.y + (Math.random() - 0.5) * 8 * a.life;
					if (i === 0) x.moveTo(p.x, jy);
					else x.lineTo(p.x, jy);
				});
				x.strokeStyle = `hsla(${a.hue},100%,70%,${a.life * 0.5})`;
				x.lineWidth = 1 + a.life * 2;
				x.shadowBlur = 15;
				x.shadowColor = `hsla(${a.hue},100%,50%,${a.life * 0.5})`;
				x.stroke();
				x.shadowBlur = 0;
			});
			arcs.filter(a => a.life > 0.05);

			const scanY = (t * 200) % h;
			x.fillStyle = 'rgba(255,0,255,0.03)';
			x.fillRect(0, scanY - 2, w, 4);

			const chrGrd = x.createLinearGradient(0, h * 0.85, 0, h);
			chrGrd.addColorStop(0, 'transparent');
			chrGrd.addColorStop(1, 'rgba(100,0,200,0.03)');
			x.fillStyle = chrGrd;
			x.fillRect(0, h * 0.85, w, h * 0.15);

			const mGrd = x.createRadialGradient(mx, my, 0, mx, my, 120);
			const mHue = (t * 50) % 360;
			mGrd.addColorStop(0, `hsla(${mHue},100%,60%,0.05)`);
			mGrd.addColorStop(1, 'transparent');
			x.fillStyle = mGrd;
			x.beginPath();
			x.arc(mx, my, 120, 0, Math.PI * 2);
			x.fill();

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#s10',
			start: 'top 80%',
			onEnter: () => {
				active = true;
				spawnArc();
				const tl = gsap.timeline({ delay: 0.3 });
				tl.to('#s10t', { opacity: 1, duration: 0.5 }, 0.8);
				tl.to('#s10s', { opacity: 1, duration: 1 }, 1.5);
			}
		});
		triggers.push(trigger);
	}

	onMount(() => {
		mx = window.innerWidth / 2;
		my = window.innerHeight / 2;

		initSupernova();
		initNeural();
		initAbyss();
		initStorm();
		initHeartbeat();
		initBlueprint();
		initPredator();
		initTimewarp();
		initVolcanic();
		initCyberpunk();
	});

	onDestroy(() => {
		triggers.forEach(t => t.kill());
		rafIds.forEach(id => cancelAnimationFrame(id));
		timeouts.forEach(id => clearTimeout(id));
		intervals.forEach(id => clearInterval(id));
	});
</script>

<svelte:window on:mousemove={handleMouseMove} />

<!-- ═══════════════════════════════════════════════════════════
     1. SUPERNOVA — Cosmic explosion, star field, gravitational
     lensing ring, nebula gas clouds, shockwave
     ═══════════════════════════════════════════════════════════ -->
<section class="hero h-supernova" id="s1">
	<canvas bind:this={c1}></canvas>
	<div class="vignette s1-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s1-content">
		<h1 class="t1" id="s1t">SUPERNOVA</h1>
		<p class="t2" id="s1s">WHEN STARS COLLAPSE, LEGENDS ARE BORN</p>
	</div>
	<span class="label">01 — SUPERNOVA</span>
	<span class="num">01</span>
</section>
<div class="divider"></div>

<!-- ═══════════════════════════════════════════════════════════
     2. NEURAL NETWORK — Synapses firing, electric pulses
     traveling along pathways, brain-like topology
     ═══════════════════════════════════════════════════════════ -->
<section class="hero h-neural" id="s2">
	<canvas bind:this={c2}></canvas>
	<div class="vignette s2-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s2-content">
		<h1 class="t1" id="s2t">SYNAPSE</h1>
		<p class="t2" id="s2s">PATTERN RECOGNITION AT NEURAL SPEED</p>
	</div>
	<span class="label">02 — NEURAL</span>
	<span class="num">02</span>
</section>
<div class="divider"></div>

<!-- ═══════════════════════════════════════════════════════════
     3. ABYSS — Deep ocean bioluminescence, pressure,
     creatures in the dark, sonar pulses
     ═══════════════════════════════════════════════════════════ -->
<section class="hero h-abyss" id="s3">
	<canvas bind:this={c3}></canvas>
	<div class="vignette s3-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s3-content">
		<h1 class="t1" id="s3t">ABYSS</h1>
		<p class="t2" id="s3s">DEPTH IS WHERE THE EDGE LIVES</p>
	</div>
	<span class="label">03 — ABYSS</span>
	<span class="num">03</span>
</section>
<div class="divider"></div>

<!-- ═══════════════════════════════════════════════════════════
     4. DIGITAL STORM — Binary tornado, electromagnetic
     interference, data corruption visuals
     ═══════════════════════════════════════════════════════════ -->
<section class="hero h-storm" id="s4">
	<canvas bind:this={c4}></canvas>
	<div class="vignette s4-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s4-content">
		<h1 class="t1" id="s4t">OVERRIDE</h1>
		<p class="t2" id="s4s">CHAOS IS THE SYSTEM RECALIBRATING</p>
	</div>
	<span class="label">04 — STORM</span>
	<span class="num">04</span>
</section>
<div class="divider"></div>

<!-- ═══════════════════════════════════════════════════════════
     5. HEARTBEAT MARKET — EKG line morphing into price chart,
     pulse-driven everything, vital signs
     ═══════════════════════════════════════════════════════════ -->
<section class="hero h-heartbeat" id="s5">
	<canvas bind:this={c5}></canvas>
	<div class="vignette s5-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s5-content">
		<h1 class="t1" id="s5t">PULSE</h1>
		<p class="t2" id="s5s">THE MARKET HAS A HEARTBEAT</p>
	</div>
	<span class="label">05 — PULSE</span>
	<span class="num">05</span>
</section>
<div class="divider"></div>

<!-- ═══════════════════════════════════════════════════════════
     6. BLUEPRINT — Wireframe world constructing itself,
     technical drawings, measurement lines
     ═══════════════════════════════════════════════════════════ -->
<section class="hero h-blueprint" id="s6">
	<canvas bind:this={c6}></canvas>
	<div class="vignette s6-vignette"></div>
	<div class="grain"></div>
	<div class="content" id="s6-content">
		<h1 class="t1" id="s6t">ARCHITECT</h1>
		<p class="t2" id="s6s">ENGINEERED FOR PRECISION</p>
	</div>
	<span class="label">06 — BLUEPRINT</span>
	<span class="num">06</span>
</section>
<div class="divider"></div>

<!-- ═══════════════════════════════════════════════════════════
     7. PREDATOR — Thermal targeting, lock-on sequence,
     threat assessment HUD, infrared
     ═══════════════════════════════════════════════════════════ -->
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

<!-- ═══════════════════════════════════════════════════════════
     8. TIME WARP — Speed lines, time dilation distortion,
     clock fragments, relativistic effects
     ═══════════════════════════════════════════════════════════ -->
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

<!-- ═══════════════════════════════════════════════════════════
     9. VOLCANIC — Magma cracks, embers, heat shimmer,
     tectonic energy, molten glow
     ═══════════════════════════════════════════════════════════ -->
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
<div class="divider"></div>

<!-- ═══════════════════════════════════════════════════════════
     10. CYBERPUNK — Neon rain, holographic glitch,
     chrome reflections, electric arcs
     ═══════════════════════════════════════════════════════════ -->
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

<div style="height:10vh;background:#000"></div>

<style>
	:global(*) {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	:global(html) {
		overflow-x: hidden;
	}

	.hero {
		position: relative;
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
		z-index: 1;
		width: 100%;
		height: 100%;
	}

	.hero .content {
		position: relative;
		z-index: 10;
		text-align: center;
		pointer-events: none;
	}

	.hero .grain {
		position: absolute;
		inset: 0;
		z-index: 50;
		pointer-events: none;
		opacity: 0.04;
		background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 512 512' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.8' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
		background-size: 256px;
	}

	.hero .vignette {
		position: absolute;
		inset: 0;
		z-index: 40;
		pointer-events: none;
	}

	.hero .label {
		position: absolute;
		top: 25px;
		left: 30px;
		z-index: 60;
		font-size: 0.5rem;
		letter-spacing: 0.5em;
		text-transform: uppercase;
		opacity: 0.2;
	}

	.hero .num {
		position: absolute;
		bottom: 30px;
		right: 35px;
		z-index: 60;
		font-family: 'Bebas Neue', sans-serif;
		font-size: 6rem;
		line-height: 1;
		opacity: 0.03;
	}

	/* HERO-SPECIFIC PALETTES */
	.h-supernova {
		background: #020005;
	}
	.h-neural {
		background: #000508;
	}
	.h-abyss {
		background: #000205;
	}
	.h-storm {
		background: #030303;
	}
	.h-heartbeat {
		background: #050000;
	}
	.h-blueprint {
		background: #000812;
	}
	.h-predator {
		background: #020200;
	}
	.h-timewarp {
		background: #02000a;
	}
	.h-volcanic {
		background: #050200;
	}
	.h-cyber {
		background: #02000a;
	}

	/* SHARED TITLE STYLES */
	.t1 {
		font-family: 'Anton', sans-serif;
		font-size: clamp(4rem, 14vw, 12rem);
		line-height: 0.85;
		letter-spacing: 0.02em;
		opacity: 0;
	}

	.t2 {
		font-family: 'Orbitron', sans-serif;
		font-size: clamp(0.5rem, 1.2vw, 0.75rem);
		letter-spacing: 0.6em;
		margin-top: 20px;
		opacity: 0;
	}

	/* DIVIDERS */
	.divider {
		height: 5vh;
		background: #000;
		position: relative;
	}

	/* SECTION-SPECIFIC VIGNETTES */
	.s1-vignette {
		background: radial-gradient(ellipse, transparent 30%, rgba(0,0,5,0.8) 100%);
	}

	.s2-vignette {
		background: radial-gradient(ellipse, transparent 40%, rgba(0,5,8,0.8) 100%);
	}

	.s3-vignette {
		background: radial-gradient(ellipse, transparent 30%, rgba(0,2,5,0.9) 100%);
	}

	.s4-vignette {
		background: radial-gradient(ellipse, transparent 35%, rgba(3,3,3,0.8) 100%);
	}

	.s5-vignette {
		background: radial-gradient(ellipse, transparent 35%, rgba(5,0,0,0.8) 100%);
	}

	.s6-vignette {
		background: radial-gradient(ellipse, transparent 40%, rgba(0,8,18,0.7) 100%);
	}

	.s7-vignette {
		background: radial-gradient(ellipse, transparent 35%, rgba(2,2,0,0.8) 100%);
	}

	.s8-vignette {
		background: radial-gradient(ellipse, transparent 30%, rgba(2,0,10,0.8) 100%);
	}

	.s9-vignette {
		background: radial-gradient(ellipse at 50% 80%, transparent 20%, rgba(5,2,0,0.8) 100%);
	}

	.s10-vignette {
		background: radial-gradient(ellipse, transparent 35%, rgba(2,0,10,0.8) 100%);
	}

	/* SECTION-SPECIFIC TITLE COLORS */
	#s1t {
		background: linear-gradient(180deg, #fff 0%, #ff8800 40%, #ff2200 100%);
		-webkit-background-clip: text;
		background-clip: text;
		color: transparent;
	}

	#s1s {
		color: #ff8800;
	}

	#s2t {
		color: #00ddff;
		text-shadow: 0 0 60px rgba(0,200,255,0.4);
	}

	#s2s {
		color: rgba(0,200,255,0.5);
	}

	#s3t {
		color: #00ffaa;
		text-shadow: 0 0 80px rgba(0,255,170,0.3);
	}

	#s3s {
		color: rgba(0,255,170,0.4);
	}

	#s4t {
		color: #fff;
		text-shadow: 0 0 40px rgba(255,255,255,0.3);
	}

	#s4s {
		color: rgba(255,255,255,0.4);
	}

	#s5t {
		color: #ff0033;
		text-shadow: 0 0 80px rgba(255,0,50,0.4);
	}

	#s5s {
		color: rgba(255,0,50,0.4);
	}

	#s6t {
		color: #4499ff;
		text-shadow: 0 0 60px rgba(68,153,255,0.3);
	}

	#s6s {
		color: rgba(68,153,255,0.4);
	}

	#s7t {
		color: #ffcc00;
		text-shadow: 0 0 60px rgba(255,200,0,0.3);
	}

	#s7s {
		color: rgba(255,200,0,0.4);
	}

	#s8t {
		color: #cc66ff;
		text-shadow: 0 0 60px rgba(200,100,255,0.4);
	}

	#s8s {
		color: rgba(200,100,255,0.4);
	}

	#s9t {
		background: linear-gradient(180deg, #ffcc00, #ff4400, #cc0000);
		-webkit-background-clip: text;
		background-clip: text;
		color: transparent;
	}

	#s9s {
		color: rgba(255,120,0,0.5);
	}

	#s10t {
		background: linear-gradient(90deg, #ff00ff, #00ffff);
		-webkit-background-clip: text;
		background-clip: text;
		color: transparent;
		text-shadow: none;
	}

	#s10s {
		color: rgba(255,0,255,0.5);
	}

	/* SECTION LABELS */
	.h-supernova .label { color: #ff8800; }
	.h-supernova .num { color: #ff4400; }

	.h-neural .label { color: #00ccff; }
	.h-neural .num { color: #00ccff; }

	.h-abyss .label { color: #00ffaa; }
	.h-abyss .num { color: #00aa88; }

	.h-storm .label { color: #888; }

	.h-heartbeat .label, .h-heartbeat .num { color: #ff0033; }

	.h-blueprint .label, .h-blueprint .num { color: #4499ff; }

	.h-predator .label, .h-predator .num { color: #ffcc00; }

	.h-timewarp .label, .h-timewarp .num { color: #cc66ff; }

	.h-volcanic .label { color: #ff6600; }
	.h-volcanic .num { color: #ff4400; }

	.h-cyber .label { color: #ff00ff; }
	.h-cyber .num { color: #00ffff; }
</style>
