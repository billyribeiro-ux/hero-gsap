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

	// Canvas refs - 10 sections with 20 canvases total (some have multiple layers)
	let h1c1: HTMLCanvasElement, h1c2: HTMLCanvasElement, h1c3: HTMLCanvasElement;
	let h2c1: HTMLCanvasElement, h2c2: HTMLCanvasElement;
	let h3c1: HTMLCanvasElement, h3c2: HTMLCanvasElement;
	let h4c1: HTMLCanvasElement, h4c2: HTMLCanvasElement;
	let h5c1: HTMLCanvasElement, h5c2: HTMLCanvasElement;
	let h6c1: HTMLCanvasElement, h6c2: HTMLCanvasElement;
	let h7c1: HTMLCanvasElement, h7c2: HTMLCanvasElement;
	let h8c1: HTMLCanvasElement, h8c2: HTMLCanvasElement;
	let h9c1: HTMLCanvasElement, h9c2: HTMLCanvasElement;
	let h10c1: HTMLCanvasElement, h10c2: HTMLCanvasElement;

	// Animation state
	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];

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
		const s = Math.floor(dur / 0.02);
		for (let j = 0; j < s; j++) {
			const f = 1 - j / s;
			gsap.set(el, {
				x: (Math.random() - 0.5) * intensity * f,
				y: (Math.random() - 0.5) * intensity * f * 0.7,
				delay: j * 0.02
			});
		}
		gsap.set(el, { x: 0, y: 0, delay: s * 0.02 });
	}

	/* ══════════════════════════════════════════════════════════════
	   1. THE FLOOR — Trading pit pandemonium
	   ══════════════════════════════════════════════════════════════ */
	function initFloor() {
		const x1 = h1c1.getContext('2d')!;
		const x2 = h1c2.getContext('2d')!;
		const x3 = h1c3.getContext('2d')!;
		let w = h1c1.width = h1c1.parentElement!.offsetWidth;
		let h = h1c1.height = h1c1.parentElement!.offsetHeight;
		let t = 0;
		let active = false;

		const GRID = 20;
		const cols = Math.ceil(1920 / GRID);
		const rows = Math.ceil(1080 / GRID);
		const energyField: number[] = new Array(cols * rows).fill(0);

		const SYMS = ['ES', 'NQ', 'SPX', 'AAPL', 'TSLA', 'NVDA', 'GC', 'CL', 'ZB', 'VIX'];
		const tickets: { x: number; y: number; vx: number; vy: number; text: string; size: number; alpha: number; rot: number; color: string }[] = [];

		for (let i = 0; i < 300; i++) {
			const a = Math.random() * Math.PI * 2;
			const spd = Math.random() * 4 + 1;
			tickets.push({
				x: Math.random() * 1920,
				y: Math.random() * 1080,
				vx: Math.cos(a) * spd,
				vy: Math.sin(a) * spd,
				text: SYMS[Math.floor(Math.random() * SYMS.length)] + ' ' + (Math.random() > 0.5 ? 'BUY' : 'SELL'),
				size: Math.random() * 8 + 6,
				alpha: Math.random() * 0.3 + 0.05,
				rot: (Math.random() - 0.5) * 0.5,
				color: Math.random() > 0.5 ? '0,255,136' : '255,51,68'
			});
		}

		const waveData: number[] = new Array(200).fill(0);

		function resize() {
			[h1c1, h1c2, h1c3].forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = h1c1.width;
			h = h1c1.height;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.008;

			x1.fillStyle = 'rgba(2,1,0,0.05)';
			x1.fillRect(0, 0, w, h);

			const mcol = Math.floor(mx / GRID);
			const mrow = Math.floor(my / GRID);
			for (let dr = -5; dr <= 5; dr++) {
				for (let dc = -5; dc <= 5; dc++) {
					const r = mrow + dr;
					const c = mcol + dc;
					if (r >= 0 && r < rows && c >= 0 && c < cols) {
						const dist = Math.sqrt(dr * dr + dc * dc);
						energyField[r * cols + c] = Math.min(1, energyField[r * cols + c] + Math.max(0, (1 - dist / 5) * 0.1));
					}
				}
			}

			for (let i = 0; i < energyField.length; i++) {
				energyField[i] *= 0.98;
				if (energyField[i] > 0.01) {
					const c = i % cols;
					const r = Math.floor(i / cols);
					const v = energyField[i];
					x1.fillStyle = `rgba(255,${Math.floor(100 + v * 100)},0,${v * 0.15})`;
					x1.fillRect(c * GRID, r * GRID, GRID, GRID);
				}
			}

			x2.clearRect(0, 0, w, h);
			tickets.forEach(tk => {
				const dx = mx - tk.x;
				const dy = my - tk.y;
				const dist = Math.sqrt(dx * dx + dy * dy);
				if (dist < 200) {
					const force = (1 - dist / 200) * 2;
					tk.vx -= (dx / dist) * force * 0.3;
					tk.vy -= (dy / dist) * force * 0.3;
				}
				tk.vx += (Math.random() - 0.5) * 0.2;
				tk.vy += (Math.random() - 0.5) * 0.2;
				tk.vx *= 0.99;
				tk.vy *= 0.99;
				tk.x += tk.vx;
				tk.y += tk.vy;
				if (tk.x < -100) tk.x = w + 100;
				if (tk.x > w + 100) tk.x = -100;
				if (tk.y < -50) tk.y = h + 50;
				if (tk.y > h + 50) tk.y = -50;

				x2.save();
				x2.translate(tk.x, tk.y);
				x2.rotate(tk.rot + Math.sin(t + tk.x * 0.01) * 0.1);
				x2.font = `${tk.size}px "JetBrains Mono"`;
				x2.fillStyle = `rgba(${tk.color},${tk.alpha})`;
				x2.fillText(tk.text, 0, 0);
				x2.restore();
			});

			x3.clearRect(0, 0, w, h);
			waveData.shift();
			waveData.push(Math.sin(t * 15) * 30 + Math.sin(t * 23) * 20 + Math.sin(t * 7) * 40 + (Math.random() - 0.5) * 60);
			x3.beginPath();
			const baseY = h * 0.85;
			waveData.forEach((v, i) => {
				const px = (i / waveData.length) * w;
				if (i === 0) x3.moveTo(px, baseY + v);
				else x3.lineTo(px, baseY + v);
			});
			x3.strokeStyle = 'rgba(255,200,0,0.15)';
			x3.lineWidth = 2;
			x3.shadowBlur = 15;
			x3.shadowColor = 'rgba(255,200,0,0.3)';
			x3.stroke();
			x3.shadowBlur = 0;

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#h1',
			start: 'top 70%',
			onEnter: () => {
				active = true;
				const tl = gsap.timeline({ delay: 0.4 });
				tl.to('#h1f', { opacity: 0.6, duration: 0.08 }, 0.5);
				tl.to('#h1f', { opacity: 0, duration: 1 }, 0.6);
				tl.to('#h1t', { opacity: 1, duration: 0.6, ease: 'power4.out' }, 0.5);
				tl.add(() => shake(document.getElementById('h1-content')!, 25, 0.5), 0.5);
				tl.to('#h1s', { opacity: 1, duration: 0.8 }, 1.2);
				tl.to('#h1s2', { opacity: 1, duration: 0.8 }, 1.6);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   2. ALGORITHM — Code rain compiling into decisions
	   ══════════════════════════════════════════════════════════════ */
	function initAlgorithm() {
		const x1 = h2c1.getContext('2d')!;
		const x2 = h2c2.getContext('2d')!;
		let w = h2c1.width = h2c1.parentElement!.offsetWidth;
		let h = h2c1.height = h2c1.parentElement!.offsetHeight;
		let t = 0;
		let active = false;

		const CODE = ['if(momentum>threshold){', '  execute(LONG,size);', '  stopLoss=atr*1.5;', '} else if(vol<0.3){', '  wait(nextSignal);', '  scanDarkPool();', 'for(let i=0;i<bars;i++){', '  sma+=close[i]/period;', '  rsi=100-100/(1+gain/loss);', 'delta=calls-puts;', 'gamma=dDelta/dPrice;', 'vega=dPrice/dVol;', 'theta=-dV/dT;', 'hedge(portfolio,beta);', 'rebalance(weights);', 'alpha=returns-benchmark;'];
		const fontSize = 11;
		const cols = Math.ceil(1920 / fontSize);
		const drops: { y: number; speed: number; line: number }[] = [];

		for (let i = 0; i < cols; i++) {
			drops[i] = { y: Math.random() * -50, speed: 0.3 + Math.random() * 0.6, line: Math.floor(Math.random() * CODE.length) };
		}

		const treeNodes: { x1: number; y1: number; x2: number; y2: number; depth: number; progress: number; label: string }[] = [];
		const treePulses: { nodeIdx: number; progress: number }[] = [];

		function buildTree(cx: number, cy: number, depth: number, angle: number, spread: number) {
			if (depth <= 0) return;
			const len = 60 - depth * 8;
			const ex = cx + Math.cos(angle) * len;
			const ey = cy + Math.sin(angle) * len;
			treeNodes.push({ x1: cx, y1: cy, x2: ex, y2: ey, depth, progress: 0, label: depth === 3 ? '' : Math.random() > 0.5 ? 'BUY' : 'SELL' });
			buildTree(ex, ey, depth - 1, angle - spread, spread * 0.8);
			buildTree(ex, ey, depth - 1, angle + spread, spread * 0.8);
		}

		function pulse() {
			if (!active) return;
			const idx = Math.floor(Math.random() * treeNodes.length);
			treePulses.push({ nodeIdx: idx, progress: 0 });
			timeouts.push(window.setTimeout(pulse, 300 + Math.random() * 500));
		}

		function resize() {
			[h2c1, h2c2].forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = h2c1.width;
			h = h2c1.height;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.008;

			x1.fillStyle = 'rgba(0,10,5,0.06)';
			x1.fillRect(0, 0, w, h);
			x1.font = `${fontSize}px "JetBrains Mono"`;

			drops.forEach((d, i) => {
				const px = i * fontSize;
				const code = CODE[d.line];
				const charIdx = Math.floor(d.y) % ((code || '').length + 5);
				const char = code ? code[charIdx % code.length] || ' ' : ' ';
				const dist = Math.hypot(px - mx, d.y * fontSize - my);
				const glow = Math.max(0, 1 - dist / 250) * 0.4;
				x1.fillStyle = `rgba(0,255,136,${0.15 + glow})`;
				if (charIdx === 0) {
					x1.shadowBlur = 10;
					x1.shadowColor = '#00ff88';
				}
				x1.fillText(char, px, d.y * fontSize);
				x1.shadowBlur = 0;
				d.y += d.speed;
				if (d.y * fontSize > h + 20) {
					d.y = -5;
					d.speed = 0.3 + Math.random() * 0.6;
					d.line = Math.floor(Math.random() * CODE.length);
				}
			});

			x2.clearRect(0, 0, w, h);
			treeNodes.forEach(n => {
				if (n.progress <= 0) return;
				const p = Math.min(n.progress, 1);
				const mx_pos = n.x1 + (n.x2 - n.x1) * p;
				const my_pos = n.y1 + (n.y2 - n.y1) * p;
				x2.beginPath();
				x2.moveTo(n.x1, n.y1);
				x2.lineTo(mx_pos, my_pos);
				const alpha = 0.1 + n.depth * 0.08;
				x2.strokeStyle = `rgba(0,255,136,${alpha * p})`;
				x2.lineWidth = n.depth * 0.5;
				x2.stroke();
				x2.beginPath();
				x2.arc(mx_pos, my_pos, 2 + n.depth, 0, Math.PI * 2);
				x2.fillStyle = `rgba(0,255,136,${alpha * p * 1.5})`;
				x2.shadowBlur = 8;
				x2.shadowColor = 'rgba(0,255,136,0.4)';
				x2.fill();
				x2.shadowBlur = 0;
				if (n.label && p > 0.9) {
					x2.font = '8px "JetBrains Mono"';
					x2.fillStyle = n.label === 'BUY' ? 'rgba(0,255,136,0.3)' : 'rgba(255,51,68,0.3)';
					x2.fillText(n.label, mx_pos + 6, my_pos + 3);
				}
			});

			treePulses.forEach(p => {
				p.progress += 0.02;
				const n = treeNodes[p.nodeIdx];
				if (!n) return;
				const px = n.x1 + (n.x2 - n.x1) * p.progress;
				const py = n.y1 + (n.y2 - n.y1) * p.progress;
				x2.beginPath();
				x2.arc(px, py, 4, 0, Math.PI * 2);
				x2.fillStyle = `rgba(0,255,200,${1 - p.progress})`;
				x2.shadowBlur = 12;
				x2.shadowColor = '#00ffcc';
				x2.fill();
				x2.shadowBlur = 0;
			});
			treePulses.filter(p => p.progress < 1);

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#h2',
			start: 'top 70%',
			onEnter: () => {
				active = true;
				treeNodes.length = 0;
				buildTree(w * 0.7, h * 0.5, 5, -Math.PI / 2, 0.6);
				treeNodes.forEach((n, i) => {
					gsap.to(n, { progress: 1, duration: 0.6, delay: i * 0.04, ease: 'power2.out' });
				});
				pulse();
				const tl = gsap.timeline({ delay: 0.3 });
				tl.to('#h2f', { opacity: 0.15, duration: 0.05 }, 1);
				tl.to('#h2f', { opacity: 0, duration: 0.5 }, 1.1);
				tl.to('#h2t', { opacity: 1, duration: 0.5 }, 1);
				tl.to('#h2s', { opacity: 1, duration: 0.8 }, 1.5);
				tl.to('#h2s2', { opacity: 1, duration: 0.8 }, 2);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   3. DARK POOL — Sonar, iceberg reveals, hidden flow
	   ══════════════════════════════════════════════════════════════ */
	function initDarkPool() {
		const x1 = h3c1.getContext('2d')!;
		const x2 = h3c2.getContext('2d')!;
		let w = h3c1.width = h3c1.parentElement!.offsetWidth;
		let h = h3c1.height = h3c1.parentElement!.offsetHeight;
		let t = 0;
		let active = false;
		const rings: { x: number; y: number; r: number; alpha: number }[] = [];
		const icebergs: { x: number; y: number; size: number; revealed: number; volume: number; side: string }[] = [];
		const particles: { x: number; y: number; vx: number; vy: number; size: number; alpha: number }[] = [];

		for (let i = 0; i < 15; i++) {
			icebergs.push({
				x: Math.random() * w,
				y: Math.random() * h,
				size: Math.random() * 60 + 30,
				revealed: 0,
				volume: Math.floor(Math.random() * 50000 + 10000),
				side: Math.random() > 0.5 ? 'BID' : 'ASK'
			});
		}

		for (let i = 0; i < 300; i++) {
			particles.push({
				x: Math.random() * w,
				y: Math.random() * h,
				vx: (Math.random() - 0.5) * 0.3,
				vy: (Math.random() - 0.5) * 0.3,
				size: Math.random() * 2 + 0.5,
				alpha: Math.random() * 0.15 + 0.02
			});
		}

		function sonarPing() {
			if (!active) return;
			rings.push({ x: mx, y: my, r: 0, alpha: 0.5 });
			icebergs.forEach(ice => {
				const d = Math.hypot(ice.x - mx, ice.y - my);
				if (d < 300) {
					gsap.to(ice, { revealed: Math.min(1, ice.revealed + 0.3), duration: 0.8, ease: 'power2.out' });
				}
			});
			timeouts.push(window.setTimeout(sonarPing, 2000));
		}

		function resize() {
			[h3c1, h3c2].forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = h3c1.width;
			h = h3c1.height;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.006;

			x1.fillStyle = 'rgba(0,3,8,0.04)';
			x1.fillRect(0, 0, w, h);

			particles.forEach(p => {
				p.x += p.vx;
				p.y += p.vy;
				if (p.x < 0) p.x = w;
				if (p.x > w) p.x = 0;
				if (p.y < 0) p.y = h;
				if (p.y > h) p.y = 0;
				x1.beginPath();
				x1.arc(p.x, p.y, p.size, 0, Math.PI * 2);
				x1.fillStyle = `rgba(26,107,255,${p.alpha})`;
				x1.fill();
			});

			rings.forEach(r => {
				r.r += 3;
				r.alpha *= 0.97;
				x1.beginPath();
				x1.arc(r.x, r.y, r.r, 0, Math.PI * 2);
				x1.strokeStyle = `rgba(26,107,255,${r.alpha})`;
				x1.lineWidth = 2;
				x1.stroke();
				x1.beginPath();
				x1.arc(r.x, r.y, r.r * 0.8, 0, Math.PI * 2);
				x1.strokeStyle = `rgba(26,107,255,${r.alpha * 0.3})`;
				x1.lineWidth = 0.5;
				x1.stroke();
			});
			rings.filter(r => r.alpha > 0.01);

			x2.clearRect(0, 0, w, h);
			icebergs.forEach(ice => {
				if (ice.revealed <= 0.01) return;
				const a = ice.revealed;
				const tipH = ice.size * 0.15;
				x2.fillStyle = `rgba(26,107,255,${a * 0.3})`;
				x2.fillRect(ice.x - ice.size * 0.3 * a, ice.y - tipH, ice.size * 0.6 * a, tipH);
				x2.fillStyle = `rgba(26,107,255,${a * 0.08})`;
				x2.fillRect(ice.x - ice.size * 0.5 * a, ice.y, ice.size * a, ice.size * 0.8 * a);
				x2.strokeStyle = `rgba(26,107,255,${a * 0.2})`;
				x2.lineWidth = 1;
				x2.strokeRect(ice.x - ice.size * 0.5 * a, ice.y, ice.size * a, ice.size * 0.8 * a);
				x2.fillStyle = `rgba(26,107,255,${a * 0.5})`;
				x2.font = '8px "JetBrains Mono"';
				x2.fillText(`${ice.side} ${ice.volume.toLocaleString()}`, ice.x - ice.size * 0.4 * a, ice.y + ice.size * 0.4 * a);
				const grd = x2.createRadialGradient(ice.x, ice.y, 0, ice.x, ice.y, ice.size * a);
				grd.addColorStop(0, `rgba(26,107,255,${a * 0.05})`);
				grd.addColorStop(1, 'transparent');
				x2.fillStyle = grd;
				x2.fillRect(ice.x - ice.size * a, ice.y - ice.size * a, ice.size * 2 * a, ice.size * 2 * a);
			});

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#h3',
			start: 'top 70%',
			onEnter: () => {
				active = true;
				sonarPing();
				const tl = gsap.timeline({ delay: 0.5 });
				tl.to('#h3t', { opacity: 1, duration: 1, ease: 'power2.out' }, 0.8);
				tl.to('#h3s', { opacity: 1, duration: 1 }, 1.5);
				tl.to('#h3s2', { opacity: 1, duration: 0.8 }, 2.2);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   4. MOMENTUM — Price ball smashing resistance levels
	   ══════════════════════════════════════════════════════════════ */
	function initMomentum() {
		const x1 = h4c1.getContext('2d')!;
		const x2 = h4c2.getContext('2d')!;
		let w = h4c1.width = h4c1.parentElement!.offsetWidth;
		let h = h4c1.height = h4c1.parentElement!.offsetHeight;
		let t = 0;
		let active = false;

		const levels = [
			{ y: 0.7, label: 'SUPPORT $5,800', broken: false, shards: [] as { x: number; y: number; vx: number; vy: number; life: number; w: number; h: number; rot: number; rv: number }[] },
			{ y: 0.55, label: 'PIVOT $5,850', broken: false, shards: [] },
			{ y: 0.4, label: 'RESISTANCE $5,900', broken: false, shards: [] },
			{ y: 0.25, label: 'ALL-TIME HIGH $5,960', broken: false, shards: [] }
		];

		const ball = { x: 0.1, y: 0.85, vx: 0.003, vy: -0.004, trail: [] as { x: number; y: number; a: number }[] };
		const priceLine: { x: number; y: number }[] = [];

		function resize() {
			[h4c1, h4c2].forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = h4c1.width;
			h = h4c1.height;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.008;

			x1.fillStyle = 'rgba(5,2,0,0.06)';
			x1.fillRect(0, 0, w, h);

			levels.forEach(lv => {
				const ly = lv.y * h;
				if (!lv.broken) {
					x1.setLineDash([8, 4]);
					x1.strokeStyle = 'rgba(255,100,50,0.15)';
					x1.lineWidth = 2;
					x1.beginPath();
					x1.moveTo(0, ly);
					x1.lineTo(w, ly);
					x1.stroke();
					x1.setLineDash([]);
					x1.fillStyle = 'rgba(255,100,50,0.12)';
					x1.font = '9px "JetBrains Mono"';
					x1.fillText(lv.label, 20, ly - 8);
				}
				lv.shards.forEach(s => {
					s.x += s.vx;
					s.y += s.vy;
					s.vy += 0.1;
					s.life -= 0.015;
					s.rot += s.rv;
					if (s.life > 0) {
						x1.save();
						x1.translate(s.x, s.y);
						x1.rotate(s.rot);
						x1.fillStyle = `rgba(255,150,50,${s.life * 0.6})`;
						x1.fillRect(-s.w / 2, -s.h / 2, s.w, s.h);
						x1.restore();
					}
				});
				lv.shards = lv.shards.filter(s => s.life > 0);
			});

			x2.clearRect(0, 0, w, h);

			ball.vy += -0.00008;
			ball.vx += 0.0001;
			ball.x += ball.vx;
			ball.y += ball.vy;
			if (ball.x > 1.1) {
				ball.x = 0.1;
				ball.y = 0.85;
				ball.vx = 0.003;
				ball.vy = -0.004;
				levels.forEach(l => { l.broken = false; });
			}

			const bx = ball.x * w;
			const by = ball.y * h;
			ball.trail.push({ x: bx, y: by, a: 1 });
			if (ball.trail.length > 80) ball.trail.shift();
			priceLine.push({ x: bx, y: by });
			if (priceLine.length > 500) priceLine.shift();

			levels.forEach(lv => {
				if (!lv.broken && Math.abs(ball.y - lv.y) < 0.02) {
					lv.broken = true;
					ball.vy *= 1.3;
					for (let i = 0; i < 40; i++) {
						lv.shards.push({
							x: bx + (Math.random() - 0.5) * 200,
							y: lv.y * h,
							vx: (Math.random() - 0.5) * 8,
							vy: (Math.random() - 0.5) * 6,
							w: Math.random() * 15 + 3,
							h: Math.random() * 4 + 1,
							life: 1,
							rot: 0,
							rv: (Math.random() - 0.5) * 0.2
						});
					}
				}
			});

			if (priceLine.length > 1) {
				x2.beginPath();
				priceLine.forEach((p, i) => {
					if (i === 0) x2.moveTo(p.x, p.y);
					else x2.lineTo(p.x, p.y);
				});
				x2.strokeStyle = 'rgba(0,255,136,0.2)';
				x2.lineWidth = 1.5;
				x2.stroke();
			}

			ball.trail.forEach((p, i) => {
				p.a -= 0.015;
				x2.beginPath();
				x2.arc(p.x, p.y, (i / ball.trail.length) * 8, 0, Math.PI * 2);
				x2.fillStyle = `rgba(0,255,136,${p.a * 0.4})`;
				x2.fill();
			});
			ball.trail = ball.trail.filter(p => p.a > 0);

			x2.beginPath();
			x2.arc(bx, by, 10, 0, Math.PI * 2);
			x2.fillStyle = '#00ff88';
			x2.shadowBlur = 30;
			x2.shadowColor = '#00ff88';
			x2.fill();
			x2.beginPath();
			x2.arc(bx, by, 5, 0, Math.PI * 2);
			x2.fillStyle = '#fff';
			x2.fill();
			x2.shadowBlur = 0;

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#h4',
			start: 'top 70%',
			onEnter: () => {
				active = true;
				const tl = gsap.timeline({ delay: 0.5 });
				tl.to('#h4t', { opacity: 1, duration: 0.5 }, 1.5);
				tl.add(() => shake(document.getElementById('h4-content')!, 20, 0.4), 1.5);
				tl.to('#h4s', { opacity: 1, duration: 1 }, 2.2);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   5. THE SPREAD — Bid vs Ask particle warfare
	   ══════════════════════════════════════════════════════════════ */
	function initSpread() {
		const x1 = h5c1.getContext('2d')!;
		const x2 = h5c2.getContext('2d')!;
		let w = h5c1.width = h5c1.parentElement!.offsetWidth;
		let h = h5c1.height = h5c1.parentElement!.offsetHeight;
		let t = 0;
		let active = false;
		const bids: { x: number; y: number; vx: number; vy: number; size: number; life: number }[] = [];
		const asks: { x: number; y: number; vx: number; vy: number; size: number; life: number }[] = [];
		const sparks: { x: number; y: number; vx: number; vy: number; life: number; color: string }[] = [];

		function spawnArmies() {
			for (let i = 0; i < 200; i++) {
				bids.push({ x: Math.random() * w * 0.35, y: Math.random() * h, vx: 1 + Math.random() * 2, vy: (Math.random() - 0.5) * 0.5, size: Math.random() * 3 + 1.5, life: 1 });
				asks.push({ x: w * 0.65 + Math.random() * w * 0.35, y: Math.random() * h, vx: -(1 + Math.random() * 2), vy: (Math.random() - 0.5) * 0.5, size: Math.random() * 3 + 1.5, life: 1 });
			}
		}

		function resize() {
			[h5c1, h5c2].forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = h5c1.width;
			h = h5c1.height;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.01;

			x1.fillStyle = 'rgba(0,0,0,0.06)';
			x1.fillRect(0, 0, w, h);
			x2.clearRect(0, 0, w, h);

			const mid = w / 2 + (mx / w - 0.5) * 100;

			x2.strokeStyle = 'rgba(255,255,255,0.05)';
			x2.setLineDash([4, 8]);
			x2.lineWidth = 1;
			x2.beginPath();
			x2.moveTo(mid, 0);
			x2.lineTo(mid, h);
			x2.stroke();
			x2.setLineDash([]);

			bids.forEach(b => {
				b.x += b.vx;
				b.y += b.vy;
				if (b.x > mid - 10) {
					b.vx *= -0.5;
					b.x = mid - 10 - Math.random() * 20;
					sparks.push({ x: mid, y: b.y, vx: (Math.random() - 0.5) * 6, vy: (Math.random() - 0.5) * 6, life: 1, color: '255,255,200' });
				}
				if (b.x < -20) { b.x = -20; b.vx = 1 + Math.random() * 2; }
				if (b.y < 0 || b.y > h) b.vy *= -1;
				x1.beginPath();
				x1.arc(b.x, b.y, b.size, 0, Math.PI * 2);
				x1.fillStyle = `rgba(0,255,136,${0.4 + Math.sin(t * 5 + b.y * 0.01) * 0.1})`;
				x1.fill();
			});

			asks.forEach(a => {
				a.x += a.vx;
				a.y += a.vy;
				if (a.x < mid + 10) {
					a.vx *= -0.5;
					a.x = mid + 10 + Math.random() * 20;
					sparks.push({ x: mid, y: a.y, vx: (Math.random() - 0.5) * 6, vy: (Math.random() - 0.5) * 6, life: 1, color: '255,255,200' });
				}
				if (a.x > w + 20) { a.x = w + 20; a.vx = -(1 + Math.random() * 2); }
				if (a.y < 0 || a.y > h) a.vy *= -1;
				x1.beginPath();
				x1.arc(a.x, a.y, a.size, 0, Math.PI * 2);
				x1.fillStyle = `rgba(255,51,68,${0.4 + Math.sin(t * 5 + a.y * 0.01) * 0.1})`;
				x1.fill();
			});

			sparks.forEach(s => {
				s.x += s.vx;
				s.y += s.vy;
				s.vx *= 0.96;
				s.vy *= 0.96;
				s.life -= 0.03;
				x2.beginPath();
				x2.arc(s.x, s.y, 2 * s.life, 0, Math.PI * 2);
				x2.fillStyle = `rgba(${s.color},${s.life})`;
				x2.shadowBlur = 6;
				x2.shadowColor = `rgba(${s.color},0.5)`;
				x2.fill();
				x2.shadowBlur = 0;
			});
			sparks.filter(s => s.life > 0);

			const clashGrd = x2.createRadialGradient(mid, h / 2, 0, mid, h / 2, 100);
			clashGrd.addColorStop(0, `rgba(255,255,200,${0.02 + Math.sin(t * 8) * 0.01})`);
			clashGrd.addColorStop(1, 'transparent');
			x2.fillStyle = clashGrd;
			x2.fillRect(mid - 100, h / 2 - 100, 200, 200);

			x2.font = '9px "JetBrains Mono"';
			x2.fillStyle = 'rgba(0,255,136,0.15)';
			x2.fillText('BIDS', 30, 30);
			x2.fillStyle = 'rgba(255,51,68,0.15)';
			x2.textAlign = 'right';
			x2.fillText('ASKS', w - 30, 30);
			x2.textAlign = 'start';

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#h5',
			start: 'top 70%',
			onEnter: () => {
				active = true;
				spawnArmies();
				const tl = gsap.timeline({ delay: 0.3 });
				tl.to('#h5t', { opacity: 1, duration: 0.5 }, 1);
				tl.add(() => shake(document.getElementById('h5-content')!, 15, 0.3), 1);
				tl.to('#h5s', { opacity: 1, duration: 1 }, 1.5);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   6. GAMMA SQUEEZE — Options chain reaction
	   ══════════════════════════════════════════════════════════════ */
	function initGamma() {
		const x1 = h6c1.getContext('2d')!;
		const x2 = h6c2.getContext('2d')!;
		let w = h6c1.width = h6c1.parentElement!.offsetWidth;
		let h = h6c1.height = h6c1.parentElement!.offsetHeight;
		let t = 0;
		let active = false;
		let rampY = 0;

		const chain: { strike: number; x: number; y: number; gamma: number; lit: number; particles: { x: number; y: number; vx: number; vy: number; life: number; size: number }[] }[] = [];

		for (let i = 0; i < 25; i++) {
			chain.push({
				strike: 5800 + i * 5,
				x: w * 0.15 + i * (w * 0.7 / 25),
				y: h * 0.5,
				gamma: Math.exp(-Math.pow(i - 12, 2) / 20) * 0.8,
				lit: 0,
				particles: []
			});
		}

		function detonateStrike(idx: number) {
			if (idx >= chain.length || idx < 0) return;
			const s = chain[idx];
			if (s.lit > 0.5) return;
			gsap.to(s, { lit: 1, duration: 0.3, ease: 'power4.out' });
			for (let i = 0; i < 30; i++) {
				const a = Math.random() * Math.PI * 2;
				const spd = Math.random() * 5 + 2;
				s.particles.push({ x: s.x, y: s.y, vx: Math.cos(a) * spd, vy: Math.sin(a) * spd, life: 1, size: Math.random() * 3 + 1 });
			}
			timeouts.push(window.setTimeout(() => detonateStrike(idx + 1), 80));
			timeouts.push(window.setTimeout(() => detonateStrike(idx - 1), 120));
		}

		function resize() {
			[h6c1, h6c2].forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = h6c1.width;
			h = h6c1.height;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.008;

			x1.fillStyle = 'rgba(10,0,5,0.06)';
			x1.fillRect(0, 0, w, h);
			x2.clearRect(0, 0, w, h);

			x1.beginPath();
			chain.forEach((s, i) => {
				const gy = h * 0.5 - s.gamma * h * 0.3 * (1 + s.lit * 0.5);
				if (i === 0) x1.moveTo(s.x, gy);
				else x1.lineTo(s.x, gy);
			});
			x1.strokeStyle = 'rgba(255,0,200,0.15)';
			x1.lineWidth = 2;
			x1.shadowBlur = 10;
			x1.shadowColor = 'rgba(255,0,200,0.3)';
			x1.stroke();
			x1.shadowBlur = 0;

			chain.forEach(s => {
				const barH = s.gamma * h * 0.25 * (1 + s.lit * 2);
				const glow = s.lit;
				x2.fillStyle = `rgba(255,${Math.floor(glow * 100)},${Math.floor(200 - glow * 100)},${0.1 + glow * 0.4})`;
				x2.fillRect(s.x - 3, h * 0.5 - barH, 6, barH);
				if (glow > 0.3) {
					x2.shadowBlur = 20;
					x2.shadowColor = `rgba(255,0,200,${glow * 0.5})`;
					x2.fillRect(s.x - 3, h * 0.5 - barH, 6, barH);
					x2.shadowBlur = 0;
				}
				s.particles.forEach(p => {
					p.x += p.vx;
					p.y += p.vy;
					p.vy += 0.05;
					p.life -= 0.02;
					if (p.life > 0) {
						x2.beginPath();
						x2.arc(p.x, p.y, p.size * p.life, 0, Math.PI * 2);
						x2.fillStyle = `rgba(255,100,200,${p.life * 0.6})`;
						x2.fill();
					}
				});
				s.particles = s.particles.filter(p => p.life > 0);
			});

			if (rampY > 0) {
				const rampPath: { x: number; y: number }[] = [];
				for (let i = 0; i < w; i += 5) {
					const prog = i / w;
					const ry = h * 0.8 - Math.pow(prog, 3) * rampY * h * 0.6;
					rampPath.push({ x: i, y: ry });
				}
				x2.beginPath();
				rampPath.forEach((p, i) => {
					if (i === 0) x2.moveTo(p.x, p.y);
					else x2.lineTo(p.x, p.y);
				});
				x2.strokeStyle = 'rgba(255,100,0,0.3)';
				x2.lineWidth = 2;
				x2.stroke();
			}

			const greeks = ['Γ', 'Δ', 'θ', 'Σ', 'Ω'];
			greeks.forEach((g, i) => {
				const gx = w * 0.2 + i * w * 0.15 + Math.sin(t + i) * 20;
				const gy = h * 0.15 + Math.cos(t * 0.7 + i) * 15;
				x2.fillStyle = `rgba(255,0,200,${0.06 + Math.sin(t * 2 + i) * 0.02})`;
				x2.font = 'bold 40px serif';
				x2.fillText(g, gx, gy);
			});

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#h6',
			start: 'top 70%',
			onEnter: () => {
				active = true;
				timeouts.push(window.setTimeout(() => detonateStrike(12), 1000));
				gsap.to({ v: 0 }, { v: 1, duration: 3, delay: 1.5, ease: 'power3.in', onUpdate: function() { rampY = this.targets()[0].v; } });
				const tl = gsap.timeline({ delay: 0.3 });
				tl.to('#h6f', { opacity: 0.3, duration: 0.05 }, 2);
				tl.to('#h6f', { opacity: 0, duration: 0.8 }, 2.1);
				tl.to('#h6t', { opacity: 1, duration: 0.5 }, 2);
				tl.add(() => shake(document.getElementById('h6-content')!, 25, 0.5), 2);
				tl.to('#h6s', { opacity: 1, duration: 0.8 }, 2.5);
				tl.to('#h6s2', { opacity: 1, duration: 0.8 }, 3);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   7. TAPE READER — Time & sales waterfall
	   ══════════════════════════════════════════════════════════════ */
	function initTape() {
		const x1 = h7c1.getContext('2d')!;
		const x2 = h7c2.getContext('2d')!;
		let w = h7c1.width = h7c1.parentElement!.offsetWidth;
		let h = h7c1.height = h7c1.parentElement!.offsetHeight;
		let t = 0;
		let active = false;
		const tapeEntries: { col: number; price: string; size: number; side: boolean; y: number; speed: number; alpha: number; isBlock: boolean; glow: number }[] = [];
		const waves: { y: number; alpha: number; x: number }[] = [];

		function addTape() {
			if (!active) return;
			const isBlock = Math.random() > 0.92;
			const size = isBlock ? Math.floor(Math.random() * 5000 + 1000) : Math.floor(Math.random() * 50 + 1);
			const price = (5890 + Math.random() * 20 - 10).toFixed(2);
			const side = Math.random() > 0.5;
			const col = Math.floor(Math.random() * 5);
			tapeEntries.push({
				col,
				price,
				size,
				side,
				y: -20,
				speed: 2 + Math.random() * 3,
				alpha: isBlock ? 0.8 : 0.3,
				isBlock,
				glow: isBlock ? 1 : 0
			});
			if (isBlock) {
				waves.push({ y: 0, alpha: 0.3, x: col * (w / 5) + w / 10 });
			}
			timeouts.push(window.setTimeout(addTape, isBlock ? 500 : 30 + Math.random() * 50));
		}

		function resize() {
			[h7c1, h7c2].forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = h7c1.width;
			h = h7c1.height;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.01;

			x1.fillStyle = 'rgba(0,0,0,0.08)';
			x1.fillRect(0, 0, w, h);
			x2.clearRect(0, 0, w, h);

			for (let i = 1; i < 5; i++) {
				x2.strokeStyle = 'rgba(0,255,136,0.03)';
				x2.lineWidth = 0.5;
				x2.beginPath();
				x2.moveTo(i * w / 5, 0);
				x2.lineTo(i * w / 5, h);
				x2.stroke();
			}

			tapeEntries.forEach(e => {
				e.y += e.speed;
				const ex = e.col * (w / 5) + 10;
				const color = e.side ? '0,255,136' : '255,51,68';
				const a = e.alpha * (1 - e.y / h * 0.5);

				if (e.isBlock) {
					x1.fillStyle = `rgba(${color},${a * 0.08})`;
					x1.fillRect(ex - 5, e.y - 8, w / 5 - 10, 18);
					e.glow *= 0.99;
				}

				x2.font = `${e.isBlock ? 'bold 11' : '10'}px "JetBrains Mono"`;
				x2.fillStyle = `rgba(${color},${a})`;
				x2.fillText(`${e.price}`, ex, e.y);
				x2.fillText(`${e.size}`, ex + 80, e.y);

				if (e.isBlock) {
					x2.shadowBlur = 10;
					x2.shadowColor = `rgba(${color},0.5)`;
					x2.fillText('■ BLOCK', ex + 140, e.y);
					x2.shadowBlur = 0;
				}
			});
			tapeEntries.filter(e => e.y < h + 50);

			waves.forEach(wv => {
				wv.alpha *= 0.97;
				x2.strokeStyle = `rgba(255,255,255,${wv.alpha})`;
				x2.lineWidth = 1;
				x2.beginPath();
				x2.moveTo(0, wv.y);
				x2.lineTo(w, wv.y);
				x2.stroke();
				wv.y += 5;
			});
			waves.filter(wv => wv.alpha > 0.01);

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#h7',
			start: 'top 70%',
			onEnter: () => {
				active = true;
				addTape();
				const tl = gsap.timeline({ delay: 0.5 });
				tl.to('#h7t', { opacity: 1, duration: 0.5 }, 1);
				tl.to('#h7s', { opacity: 1, duration: 1 }, 1.5);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   8. FIBONACCI — Golden spiral constructing
	   ══════════════════════════════════════════════════════════════ */
	function initFibonacci() {
		const x1 = h8c1.getContext('2d')!;
		const x2 = h8c2.getContext('2d')!;
		let w = h8c1.width = h8c1.parentElement!.offsetWidth;
		let h = h8c1.height = h8c1.parentElement!.offsetHeight;
		let t = 0;
		let active = false;
		let spiralProgress = 0;

		const PHI = 1.618033988749;
		const LEVELS = [0.236, 0.382, 0.5, 0.618, 0.786];

		function resize() {
			[h8c1, h8c2].forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = h8c1.width;
			h = h8c1.height;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.005;

			x1.fillStyle = 'rgba(2,0,8,0.03)';
			x1.fillRect(0, 0, w, h);
			x2.clearRect(0, 0, w, h);

			const cx = w * 0.45 + (mx / w - 0.5) * 30;
			const cy = h * 0.55 + (my / h - 0.5) * 30;

			if (spiralProgress > 0) {
				x1.beginPath();
				let r = 5;
				const maxAngle = spiralProgress * Math.PI * 8;
				for (let a = 0; a < maxAngle; a += 0.02) {
					r = 5 * Math.pow(PHI, a / (Math.PI * 2));
					const px = cx + Math.cos(a + t * 0.3) * r;
					const py = cy + Math.sin(a + t * 0.3) * r;
					if (a === 0) x1.moveTo(px, py);
					else x1.lineTo(px, py);
				}
				x1.strokeStyle = `rgba(255,215,0,${0.15 + Math.sin(t * 3) * 0.05})`;
				x1.lineWidth = 1.5;
				x1.shadowBlur = 10;
				x1.shadowColor = 'rgba(255,215,0,0.3)';
				x1.stroke();
				x1.shadowBlur = 0;

				let rw = 30 * spiralProgress;
				let rh = rw / PHI;
				for (let i = 0; i < 6 && i < spiralProgress * 8; i++) {
					x2.save();
					x2.translate(cx, cy);
					x2.rotate(i * Math.PI / 2 + t * 0.1);
					x2.strokeStyle = `rgba(255,215,0,${0.04 * (6 - i) / 6})`;
					x2.lineWidth = 0.5;
					x2.strokeRect(-rw / 2, -rh / 2, rw, rh);
					x2.restore();
					const tmp = rw;
					rw = rw + rh;
					rh = tmp;
				}
			}

			LEVELS.forEach((lv, i) => {
				const ly = h * 0.15 + lv * h * 0.7;
				const alpha = 0.05 + Math.sin(t * 2 + i) * 0.02;
				x2.setLineDash([2, 6]);
				x2.strokeStyle = `rgba(255,215,0,${alpha})`;
				x2.lineWidth = 0.5;
				x2.beginPath();
				x2.moveTo(w * 0.1, ly);
				x2.lineTo(w * 0.9, ly);
				x2.stroke();
				x2.setLineDash([]);
				x2.fillStyle = `rgba(255,215,0,${alpha * 2})`;
				x2.font = '8px "JetBrains Mono"';
				x2.fillText(lv.toFixed(3), w * 0.91, ly + 3);

				const dist = Math.abs(my - ly);
				if (dist < 40) {
					const grd = x2.createLinearGradient(0, ly - 20, 0, ly + 20);
					grd.addColorStop(0, 'transparent');
					grd.addColorStop(0.5, `rgba(255,215,0,${(1 - dist / 40) * 0.08})`);
					grd.addColorStop(1, 'transparent');
					x2.fillStyle = grd;
					x2.fillRect(0, ly - 20, w, 40);
				}
			});

			const ratios = ['1.618', '0.618', '2.618', '0.382', '1.000'];
			ratios.forEach((r, i) => {
				const rx = w * 0.1 + i * w * 0.2 + Math.sin(t * 0.5 + i) * 20;
				const ry = h * 0.9 + Math.cos(t * 0.7 + i) * 10;
				x2.fillStyle = `rgba(255,215,0,${0.05 + Math.sin(t + i) * 0.02})`;
				x2.font = '18px Orbitron';
				x2.fillText(r, rx, ry);
			});

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#h8',
			start: 'top 70%',
			onEnter: () => {
				active = true;
				gsap.to({ v: 0 }, { v: 1, duration: 4, ease: 'power2.out', onUpdate: function() { spiralProgress = this.targets()[0].v; } });
				const tl = gsap.timeline({ delay: 0.5 });
				tl.to('#h8t', { opacity: 1, duration: 1, ease: 'power2.out' }, 1.5);
				tl.to('#h8s', { opacity: 1, duration: 1 }, 2.5);
				tl.to('#h8s2', { opacity: 1, duration: 0.8 }, 3.2);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   9. VOLATILITY STORM — VIX weather system
	   ══════════════════════════════════════════════════════════════ */
	function initVolatility() {
		const x1 = h9c1.getContext('2d')!;
		const x2 = h9c2.getContext('2d')!;
		let w = h9c1.width = h9c1.parentElement!.offsetWidth;
		let h = h9c1.height = h9c1.parentElement!.offsetHeight;
		let t = 0;
		let active = false;
		const clouds: { x: number; y: number; r: number; vx: number; darkness: number }[] = [];
		const bolts: { pts: { x: number; y: number; branch?: boolean; resume?: boolean }[]; life: number }[] = [];
		const rain: { x: number; y: number; speed: number; len: number; alpha: number }[] = [];

		for (let i = 0; i < 20; i++) {
			clouds.push({
				x: Math.random() * w,
				y: Math.random() * h * 0.4,
				r: Math.random() * 120 + 60,
				vx: (Math.random() - 0.5) * 0.5,
				darkness: Math.random() * 0.3 + 0.1
			});
		}

		for (let i = 0; i < 400; i++) {
			rain.push({
				x: Math.random() * w,
				y: Math.random() * h,
				speed: Math.random() * 8 + 4,
				len: Math.random() * 20 + 10,
				alpha: Math.random() * 0.2 + 0.05
			});
		}

		function lightning() {
			if (!active) return;
			const pts: { x: number; y: number; branch?: boolean; resume?: boolean }[] = [{ x: Math.random() * w * 0.6 + w * 0.2, y: 0 }];
			let y = 0;
			while (y < h * 0.7) {
				y += Math.random() * 40 + 20;
				const lastX = pts[pts.length - 1].x;
				pts.push({ x: lastX + (Math.random() - 0.5) * 80, y });
				if (Math.random() > 0.6) {
					let bx = lastX;
					let by = y;
					for (let i = 0; i < 3; i++) {
						bx += (Math.random() - 0.5) * 40;
						by += Math.random() * 30 + 10;
						pts.push({ x: bx, y: by, branch: true });
					}
					pts.push({ x: lastX + (Math.random() - 0.5) * 80, y: y + 10, resume: true });
				}
			}
			bolts.push({ pts, life: 1 });
			gsap.to('#h9f', { opacity: 0.5, duration: 0.03, onComplete: () => gsap.to('#h9f', { opacity: 0, duration: 0.4 }) });
			shake(document.getElementById('h9-content')!, 10, 0.2);
			timeouts.push(window.setTimeout(lightning, Math.random() * 3000 + 1000));
		}

		function resize() {
			[h9c1, h9c2].forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = h9c1.width;
			h = h9c1.height;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.006;

			x1.fillStyle = 'rgba(3,0,2,0.08)';
			x1.fillRect(0, 0, w, h);
			x2.clearRect(0, 0, w, h);

			clouds.forEach(cl => {
				cl.x += cl.vx;
				if (cl.x < -cl.r) cl.x = w + cl.r;
				if (cl.x > w + cl.r) cl.x = -cl.r;
				const grd = x1.createRadialGradient(cl.x, cl.y, 0, cl.x, cl.y, cl.r);
				grd.addColorStop(0, `rgba(40,10,20,${cl.darkness})`);
				grd.addColorStop(1, 'transparent');
				x1.fillStyle = grd;
				x1.fillRect(cl.x - cl.r, cl.y - cl.r, cl.r * 2, cl.r * 2);
			});

			rain.forEach(r => {
				r.y += r.speed;
				r.x -= 1;
				if (r.y > h) {
					r.y = -r.len;
					r.x = Math.random() * w;
				}
				x1.beginPath();
				x1.moveTo(r.x, r.y);
				x1.lineTo(r.x - 2, r.y + r.len);
				x1.strokeStyle = `rgba(180,130,140,${r.alpha})`;
				x1.lineWidth = 0.5;
				x1.stroke();
			});

			bolts.forEach(b => {
				b.life *= 0.9;
				x2.beginPath();
				let branching = false;
				b.pts.forEach((p, i) => {
					if (p.resume) {
						branching = false;
						return;
					}
					if (p.branch && !branching) {
						x2.stroke();
						x2.beginPath();
						x2.moveTo(p.x, p.y);
						branching = true;
						return;
					}
					const jx = p.x + (Math.random() - 0.5) * 4 * b.life;
					const jy = p.y + (Math.random() - 0.5) * 2 * b.life;
					if (i === 0 || p.resume) x2.moveTo(jx, jy);
					else x2.lineTo(jx, jy);
				});
				x2.strokeStyle = `rgba(200,180,255,${b.life * 0.8})`;
				x2.lineWidth = 1 + b.life * 3;
				x2.shadowBlur = 20;
				x2.shadowColor = `rgba(200,150,255,${b.life})`;
				x2.stroke();
				x2.shadowBlur = 0;
			});
			bolts.filter(b => b.life > 0.05);

			const vix = 18 + Math.sin(t * 0.8) * 12 + Math.sin(t * 3) * 5;
			const gaugeX = w - 120;
			const gaugeY = h - 80;
			x2.beginPath();
			x2.arc(gaugeX, gaugeY, 50, -Math.PI, 0);
			x2.strokeStyle = 'rgba(255,34,68,0.15)';
			x2.lineWidth = 4;
			x2.stroke();
			const vixAngle = -Math.PI + ((vix - 5) / 50) * Math.PI;
			x2.beginPath();
			x2.moveTo(gaugeX, gaugeY);
			x2.lineTo(gaugeX + Math.cos(vixAngle) * 45, gaugeY + Math.sin(vixAngle) * 45);
			x2.strokeStyle = 'rgba(255,34,68,0.6)';
			x2.lineWidth = 2;
			x2.stroke();
			x2.fillStyle = 'rgba(255,34,68,0.4)';
			x2.font = 'bold 18px Orbitron';
			x2.textAlign = 'center';
			x2.fillText(vix.toFixed(1), gaugeX, gaugeY + 25);
			x2.font = '8px "JetBrains Mono"';
			x2.fillText('VIX', gaugeX, gaugeY + 38);
			x2.textAlign = 'start';

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#h9',
			start: 'top 70%',
			onEnter: () => {
				active = true;
				timeouts.push(window.setTimeout(lightning, 800));
				const tl = gsap.timeline({ delay: 0.5 });
				tl.to('#h9t', { opacity: 1, duration: 0.4 }, 1);
				tl.add(() => shake(document.getElementById('h9-content')!, 20, 0.4), 1);
				tl.to('#h9s', { opacity: 1, duration: 1 }, 1.5);
				tl.to('#h9s2', { opacity: 1, duration: 0.8 }, 2.2);
			}
		});
		triggers.push(trigger);
	}

	/* ══════════════════════════════════════════════════════════════
	   10. THE EDGE — Probability distributions
	   ══════════════════════════════════════════════════════════════ */
	function initEdge() {
		const x1 = h10c1.getContext('2d')!;
		const x2 = h10c2.getContext('2d')!;
		let w = h10c1.width = h10c1.parentElement!.offsetWidth;
		let h = h10c1.height = h10c1.parentElement!.offsetHeight;
		let t = 0;
		let active = false;
		let evLine = 0;

		const dataPoints: { x: number; y: number; ox: number; oy: number; vx: number; vy: number; size: number; alpha: number; dist: number }[] = [];

		for (let i = 0; i < 500; i++) {
			const u1 = Math.random();
			const u2 = Math.random();
			const z = Math.sqrt(-2 * Math.log(u1)) * Math.cos(2 * Math.PI * u2);
			const z2 = Math.sqrt(-2 * Math.log(u1)) * Math.sin(2 * Math.PI * u2);
			dataPoints.push({
				x: w / 2 + z * w * 0.15,
				y: h / 2 + z2 * h * 0.15,
				ox: w / 2 + z * w * 0.15,
				oy: h / 2 + z2 * h * 0.15,
				vx: 0,
				vy: 0,
				size: Math.random() * 2.5 + 0.5,
				alpha: Math.random() * 0.3 + 0.1,
				dist: Math.sqrt(z * z + z2 * z2)
			});
		}

		function resize() {
			[h10c1, h10c2].forEach(c => {
				c.width = c.parentElement!.offsetWidth;
				c.height = c.parentElement!.offsetHeight;
			});
			w = h10c1.width;
			h = h10c1.height;
		}
		window.addEventListener('resize', resize);

		function draw() {
			if (!active) {
				rafIds.push(requestAnimationFrame(draw));
				return;
			}
			t += 0.006;

			x1.fillStyle = 'rgba(0,2,8,0.04)';
			x1.fillRect(0, 0, w, h);
			x2.clearRect(0, 0, w, h);

			const cx = w / 2;
			const cy = h / 2;

			x2.beginPath();
			for (let i = 0; i < w; i += 3) {
				const z = (i - cx) / (w * 0.15);
				const bell = Math.exp(-z * z / 2) * h * 0.35;
				if (i === 0) x2.moveTo(i, cy - bell);
				else x2.lineTo(i, cy - bell);
			}
			x2.strokeStyle = `rgba(0,200,255,${0.06 + Math.sin(t * 2) * 0.02})`;
			x2.lineWidth = 1;
			x2.stroke();

			x2.beginPath();
			for (let i = 0; i < w; i += 3) {
				const z = (i - cx) / (w * 0.15);
				const bell = Math.exp(-z * z / 2) * h * 0.35;
				if (i === 0) x2.moveTo(i, cy + bell);
				else x2.lineTo(i, cy + bell);
			}
			x2.strokeStyle = 'rgba(0,200,255,0.03)';
			x2.stroke();

			[1, 2, 3].forEach(sd => {
				const xPos = cx + sd * w * 0.15;
				const xNeg = cx - sd * w * 0.15;
				x2.strokeStyle = `rgba(0,200,255,${0.04 - sd * 0.01})`;
				x2.setLineDash([2, 4]);
				x2.lineWidth = 0.5;
				[xPos, xNeg].forEach(px => {
					x2.beginPath();
					x2.moveTo(px, cy - h * 0.4);
					x2.lineTo(px, cy + h * 0.4);
					x2.stroke();
				});
				x2.setLineDash([]);
				x2.fillStyle = 'rgba(0,200,255,0.04)';
				x2.font = '7px "JetBrains Mono"';
				x2.fillText(`${sd}σ`, xPos + 3, cy - h * 0.35);
				x2.fillText(`-${sd}σ`, xNeg + 3, cy - h * 0.35);
			});

			dataPoints.forEach(p => {
				const dx = mx - p.x;
				const dy = my - p.y;
				const dist = Math.sqrt(dx * dx + dy * dy);
				if (dist < 200) {
					p.vx += (dx / dist) * 0.3 * (1 - dist / 200);
					p.vy += (dy / dist) * 0.3 * (1 - dist / 200);
				}
				p.vx += (p.ox - p.x) * 0.01;
				p.vy += (p.oy - p.y) * 0.01;
				p.vx *= 0.95;
				p.vy *= 0.95;
				p.x += p.vx;
				p.y += p.vy;

				const r = Math.min(255, p.dist * 80);
				const b = 255 - r;
				x1.beginPath();
				x1.arc(p.x, p.y, p.size, 0, Math.PI * 2);
				x1.fillStyle = `rgba(${r},${Math.floor(b * 0.7)},${b},${p.alpha})`;
				if (p.dist < 0.5) {
					x1.shadowBlur = 4;
					x1.shadowColor = 'rgba(0,200,255,0.2)';
				}
				x1.fill();
				x1.shadowBlur = 0;
			});

			if (evLine > 0) {
				x2.strokeStyle = `rgba(0,255,200,${evLine * 0.3})`;
				x2.lineWidth = 2;
				x2.setLineDash([]);
				x2.beginPath();
				x2.moveTo(cx, cy - h * 0.4 * evLine);
				x2.lineTo(cx, cy + h * 0.4 * evLine);
				x2.stroke();
				x2.shadowBlur = 15;
				x2.shadowColor = 'rgba(0,255,200,0.4)';
				x2.stroke();
				x2.shadowBlur = 0;
				x2.fillStyle = `rgba(0,255,200,${evLine * 0.5})`;
				x2.font = 'bold 10px "JetBrains Mono"';
				x2.fillText('E[V] = +', cx + 8, cy - h * 0.35 * evLine);
			}

			const wr = 0.63 + Math.sin(t * 0.5) * 0.05;
			x2.fillStyle = 'rgba(0,200,255,0.08)';
			x2.font = '40px Orbitron';
			x2.textAlign = 'center';
			x2.fillText((wr * 100).toFixed(1) + '%', 80, h - 50);
			x2.font = '8px "JetBrains Mono"';
			x2.fillText('WIN RATE', 80, h - 30);
			x2.textAlign = 'start';

			rafIds.push(requestAnimationFrame(draw));
		}
		rafIds.push(requestAnimationFrame(draw));

		const trigger = ScrollTrigger.create({
			trigger: '#h10',
			start: 'top 70%',
			onEnter: () => {
				active = true;
				gsap.to({ v: 0 }, { v: 1, duration: 2, delay: 1, ease: 'power2.out', onUpdate: function() { evLine = this.targets()[0].v; } });
				const tl = gsap.timeline({ delay: 0.5 });
				tl.to('#h10t', { opacity: 1, duration: 0.8, ease: 'power2.out' }, 1);
				tl.to('#h10s', { opacity: 1, duration: 1 }, 1.8);
				tl.to('#h10s2', { opacity: 1, duration: 0.8 }, 2.5);
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

		initFloor();
		initAlgorithm();
		initDarkPool();
		initMomentum();
		initSpread();
		initGamma();
		initTape();
		initFibonacci();
		initVolatility();
		initEdge();
	});

	onDestroy(() => {
		triggers.forEach(t => t.kill());
		rafIds.forEach(id => cancelAnimationFrame(id));
		timeouts.forEach(id => clearTimeout(id));
	});
</script>

<svelte:window on:mousemove={handleMouseMove} />

<!-- ══════════════════════════════════════════════════════════
     1. THE FLOOR — Trading pit pandemonium
     ══════════════════════════════════════════════════════════ -->
<section class="hero" id="h1" style="background:#020100">
	<canvas bind:this={h1c1}></canvas>
	<canvas bind:this={h1c2} class="c2"></canvas>
	<canvas bind:this={h1c3} class="c3"></canvas>
	<div class="vig h1-vig"></div>
	<div class="grn"></div>
	<div class="flash" id="h1f" style="background:radial-gradient(circle,rgba(255,200,50,0.8),transparent 60%)"></div>
	<div class="ct" id="h1-content">
		<h1 class="t1" id="h1t">THE FLOOR</h1>
		<p class="t2" id="h1s">WHERE FORTUNES ARE MADE IN SECONDS</p>
		<p class="t3" id="h1s2">[ OPEN OUTCRY — CHICAGO, 1987 ]</p>
	</div>
	<span class="lbl h1-lbl">01 — THE FLOOR</span>
	<span class="idx h1-idx">01</span>
</section>
<div class="sp"></div>

<!-- ══════════════════════════════════════════════════════════
     2. ALGORITHM — Living code
     ══════════════════════════════════════════════════════════ -->
<section class="hero" id="h2" style="background:#000a05">
	<canvas bind:this={h2c1}></canvas>
	<canvas bind:this={h2c2} class="c2"></canvas>
	<div class="vig h2-vig"></div>
	<div class="grn"></div>
	<div class="flash" id="h2f" style="background:#00ff88"></div>
	<div class="ct" id="h2-content">
		<h1 class="t1" id="h2t">ALGORITHM</h1>
		<p class="t2" id="h2s">LOGIC IS THE ULTIMATE WEAPON</p>
		<p class="t3" id="h2s2">[ LATENCY: 0.003ms // EXECUTING ]</p>
	</div>
	<span class="lbl h2-lbl">02 — ALGORITHM</span>
	<span class="idx h2-idx">02</span>
</section>
<div class="sp"></div>

<!-- ══════════════════════════════════════════════════════════
     3. DARK POOL — Hidden liquidity
     ══════════════════════════════════════════════════════════ -->
<section class="hero" id="h3" style="background:#000308">
	<canvas bind:this={h3c1}></canvas>
	<canvas bind:this={h3c2} class="c2"></canvas>
	<div class="vig h3-vig"></div>
	<div class="grn"></div>
	<div class="ct" id="h3-content">
		<h1 class="t1" id="h3t">DARK POOL</h1>
		<p class="t2" id="h3s">THE ORDERS YOU'LL NEVER SEE COMING</p>
		<p class="t3" id="h3s2">[ HIDDEN LIQUIDITY DETECTED ]</p>
	</div>
	<span class="lbl h3-lbl">03 — DARK POOL</span>
	<span class="idx h3-idx">03</span>
</section>
<div class="sp"></div>

<!-- ══════════════════════════════════════════════════════════
     4. MOMENTUM — Price ball smashing resistance
     ══════════════════════════════════════════════════════════ -->
<section class="hero" id="h4" style="background:#050200">
	<canvas bind:this={h4c1}></canvas>
	<canvas bind:this={h4c2} class="c2"></canvas>
	<div class="vig h4-vig"></div>
	<div class="grn"></div>
	<div class="flash" id="h4f" style="background:#fff"></div>
	<div class="ct" id="h4-content">
		<h1 class="t1" id="h4t">BREAKOUT</h1>
		<p class="t2" id="h4s">RESISTANCE IS TEMPORARY. MOMENTUM IS FOREVER.</p>
	</div>
	<span class="lbl h4-lbl">04 — MOMENTUM</span>
	<span class="idx h4-idx">04</span>
</section>
<div class="sp"></div>

<!-- ══════════════════════════════════════════════════════════
     5. THE SPREAD — Bid vs Ask particle warfare
     ══════════════════════════════════════════════════════════ -->
<section class="hero" id="h5" style="background:#000">
	<canvas bind:this={h5c1}></canvas>
	<canvas bind:this={h5c2} class="c2"></canvas>
	<div class="vig h5-vig"></div>
	<div class="grn"></div>
	<div class="flash" id="h5f" style="background:rgba(255,255,255,0.8)"></div>
	<div class="ct" id="h5-content">
		<h1 class="t1" id="h5t">THE SPREAD</h1>
		<p class="t2" id="h5s">WHERE BUYERS AND SELLERS GO TO WAR</p>
	</div>
	<span class="lbl h5-lbl">05 — THE SPREAD</span>
	<span class="idx h5-idx">05</span>
</section>
<div class="sp"></div>

<!-- ══════════════════════════════════════════════════════════
     6. GAMMA SQUEEZE — Options chain reaction
     ══════════════════════════════════════════════════════════ -->
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

<!-- ══════════════════════════════════════════════════════════
     7. TAPE READER — Time & sales waterfall
     ══════════════════════════════════════════════════════════ -->
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

<!-- ══════════════════════════════════════════════════════════
     8. FIBONACCI — Golden spiral
     ══════════════════════════════════════════════════════════ -->
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

<!-- ══════════════════════════════════════════════════════════
     9. VOLATILITY STORM — VIX weather system
     ══════════════════════════════════════════════════════════ -->
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

<!-- ══════════════════════════════════════════════════════════
     10. THE EDGE — Probability cloud
     ══════════════════════════════════════════════════════════ -->
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

<div style="height:10vh;background:#000"></div>

<style>
	:global(*) {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	:global(html) {
		overflow-x: hidden;
		background: #000;
	}

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
		z-index: 1;
		width: 100%;
		height: 100%;
	}

	.hero canvas.c2 {
		z-index: 2;
	}

	.hero canvas.c3 {
		z-index: 3;
	}

	.hero .ct {
		position: relative;
		z-index: 10;
		text-align: center;
		pointer-events: none;
	}

	.hero .vig {
		position: absolute;
		inset: 0;
		z-index: 40;
		pointer-events: none;
	}

	.hero .grn {
		position: absolute;
		inset: 0;
		z-index: 50;
		pointer-events: none;
		opacity: 0.03;
		mix-blend-mode: overlay;
		background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
		background-size: 128px;
	}

	.hero .lbl {
		position: absolute;
		top: 25px;
		left: 30px;
		z-index: 60;
		font-size: 0.45rem;
		letter-spacing: 0.6em;
		text-transform: uppercase;
		opacity: 0.15;
	}

	.hero .idx {
		position: absolute;
		bottom: 25px;
		right: 35px;
		z-index: 60;
		font-family: 'Bebas Neue';
		font-size: 5rem;
		line-height: 1;
		opacity: 0.03;
	}

	.hero .flash {
		position: absolute;
		inset: 0;
		z-index: 30;
		pointer-events: none;
		opacity: 0;
	}

	.t1 {
		font-family: 'Anton', sans-serif;
		font-size: clamp(3.5rem, 13vw, 11rem);
		line-height: 0.85;
		letter-spacing: 0.02em;
		opacity: 0;
	}

	.t2 {
		font-family: 'Orbitron', sans-serif;
		font-size: clamp(0.45rem, 1.1vw, 0.7rem);
		letter-spacing: 0.7em;
		margin-top: 18px;
		opacity: 0;
	}

	.t3 {
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.5rem;
		letter-spacing: 0.35em;
		margin-top: 28px;
		opacity: 0;
	}

	.sp {
		height: 2px;
		background: #000;
	}

	/* Section backgrounds */
	#h1 { background: #020100; }
	#h2 { background: #000a05; }
	#h3 { background: #000308; }
	#h4 { background: #050200; }
	#h5 { background: #000; }
	#h6 { background: #0a0005; }
	#h7 { background: #000; }
	#h8 { background: #020008; }
	#h9 { background: #030002; }
	#h10 { background: #000208; }

	/* Vignettes */
	.h1-vig { background: radial-gradient(ellipse at 50% 60%, transparent 20%, rgba(2,1,0,0.85) 100%); }
	.h2-vig { background: radial-gradient(ellipse, transparent 30%, rgba(0,10,5,0.85) 100%); }
	.h3-vig { background: radial-gradient(ellipse, transparent 25%, rgba(0,3,8,0.9) 100%); }
	.h4-vig { background: radial-gradient(ellipse, transparent 30%, rgba(5,2,0,0.85) 100%); }
	.h5-vig { background: radial-gradient(ellipse, transparent 35%, rgba(0,0,0,0.8) 100%); }
	.h6-vig { background: radial-gradient(ellipse, transparent 30%, rgba(10,0,5,0.85) 100%); }
	.h7-vig { background: radial-gradient(ellipse, transparent 40%, rgba(0,0,0,0.7) 100%); }
	.h8-vig { background: radial-gradient(ellipse, transparent 30%, rgba(2,0,8,0.85) 100%); }
	.h9-vig { background: radial-gradient(ellipse at 50% 40%, transparent 20%, rgba(3,0,2,0.9) 100%); }
	.h10-vig { background: radial-gradient(ellipse, transparent 35%, rgba(0,2,8,0.85) 100%); }

	/* Title colors */
	#h1t {
		background: linear-gradient(180deg, #ffd700 0%, #ff8c00 50%, #8b4513 100%);
		-webkit-background-clip: text;
		background-clip: text;
		color: transparent;
	}
	#h1s { color: #ffd700; }
	#h1s2 { color: rgba(255,200,0,0.3); }

	#h2t { color: #00ff88; text-shadow: 0 0 80px rgba(0,255,136,0.3); }
	#h2s { color: rgba(0,255,136,0.5); }
	#h2s2 { color: rgba(0,255,136,0.2); }

	#h3t { color: #1a6bff; text-shadow: 0 0 100px rgba(26,107,255,0.4); }
	#h3s { color: rgba(26,107,255,0.4); }
	#h3s2 { color: rgba(26,107,255,0.2); }

	#h4t {
		background: linear-gradient(180deg, #00ff88, #00cc66, #006633);
		-webkit-background-clip: text;
		background-clip: text;
		color: transparent;
	}
	#h4s { color: rgba(0,255,136,0.5); }

	#h5t { color: #fff; text-shadow: 0 0 60px rgba(255,255,255,0.2); }
	#h5s { color: rgba(255,255,255,0.3); }

	#h6t {
		background: linear-gradient(90deg, #ff00cc, #ff6600);
		-webkit-background-clip: text;
		background-clip: text;
		color: transparent;
	}
	#h6s { color: rgba(255,0,200,0.5); }
	#h6s2 { color: rgba(255,100,0,0.3); }

	#h7t { color: #fff; text-shadow: 0 0 40px rgba(0,255,136,0.2); }
	#h7s { color: rgba(0,255,136,0.4); }

	#h8t {
		background: linear-gradient(180deg, #ffd700, #ff8c00, #cc6600);
		-webkit-background-clip: text;
		background-clip: text;
		color: transparent;
	}
	#h8s { color: rgba(255,215,0,0.4); }
	#h8s2 { color: rgba(255,215,0,0.2); }

	#h9t { color: #ff2244; text-shadow: 0 0 100px rgba(255,34,68,0.5); }
	#h9s { color: rgba(255,34,68,0.4); }
	#h9s2 { color: rgba(255,34,68,0.2); }

	#h10t {
		background: linear-gradient(180deg, #fff, #00ddff, #0066ff);
		-webkit-background-clip: text;
		background-clip: text;
		color: transparent;
	}
	#h10s { color: rgba(0,200,255,0.4); }
	#h10s2 { color: rgba(0,200,255,0.2); }

	/* Labels and indices */
	.h1-lbl, .h1-idx { color: #ffd700; }
	.h2-lbl, .h2-idx { color: #00ff88; }
	.h3-lbl, .h3-idx { color: #1a6bff; }
	.h4-lbl, .h4-idx { color: #00ff88; }
	.h5-lbl { color: #888; }
	.h6-lbl, .h6-idx { color: #ff00cc; }
	.h7-lbl, .h7-idx { color: #00ff88; }
	.h8-lbl, .h8-idx { color: #ffd700; }
	.h9-lbl, .h9-idx { color: #ff2244; }
	.h10-lbl, .h10-idx { color: #00ddff; }
</style>
