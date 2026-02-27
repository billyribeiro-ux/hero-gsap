<script context="module" lang="ts">
	class Shard {
		x: number; y: number; w: number; h: number;
		vx: number; vy: number; rotation: number; rotSpeed: number;
		life: number; decay: number; color: string;
		constructor(x: number, y: number, cw: number, ch: number, color: string, sw: number, sh: number) {
			this.x = x; this.y = y; this.w = cw; this.h = ch; this.color = color;
			const angle = Math.atan2(y - sh / 2, x - sw / 2) + (Math.random() - 0.5) * 0.5;
			const speed = Math.random() * 20 + 10;
			this.vx = Math.cos(angle) * speed; this.vy = Math.sin(angle) * speed;
			this.rotation = 0; this.rotSpeed = (Math.random() - 0.5) * 0.3;
			this.life = 1; this.decay = 0.008 + Math.random() * 0.01;
		}
		update() {
			this.x += this.vx; this.y += this.vy;
			this.vy += 0.15; this.vx *= 0.99;
			this.rotation += this.rotSpeed; this.life -= this.decay;
		}
		draw(c: CanvasRenderingContext2D) {
			if (this.life <= 0) return;
			c.save(); c.globalAlpha = this.life;
			c.translate(this.x, this.y); c.rotate(this.rotation);
			c.fillStyle = this.color;
			c.fillRect(-this.w / 2, -this.h / 2, this.w, this.h);
			c.restore();
		}
	}
</script>

<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	let section: HTMLElement;
	let canvas: HTMLCanvasElement;
	let shtFlash: HTMLElement;
	let shtContent: HTMLElement;
	let shtRain: HTMLElement;
	let shtRule: HTMLElement;
	let shtSub: HTMLElement;

	onMount(() => {
		gsap.registerPlugin(ScrollTrigger);
		const ctx = canvas.getContext('2d')!;
		let w = (canvas.width = window.innerWidth);
		let h = (canvas.height = window.innerHeight);
		let raf: number;

		function resize() { w = canvas.width = window.innerWidth; h = canvas.height = window.innerHeight; }
		window.addEventListener('resize', resize);

		type Candle = { open: number; close: number; high: number; low: number; bull: boolean };
		let candles: Candle[] = [];

		function genCandles() {
			candles = [];
			let price = 150;
			for (let i = 0; i < 40; i++) {
				const open = price;
				const vol = 3 + Math.random() * 8;
				const close = price + (Math.random() - 0.42) * vol;
				const high = Math.max(open, close) + Math.random() * vol * 0.5;
				const low = Math.min(open, close) - Math.random() * vol * 0.5;
				candles.push({ open, close, high, low, bull: close >= open });
				price = close;
			}
		}
		genCandles();


		let shards: Shard[] = [];
		let phase: 'build' | 'shatter' | 'done' = 'build';
		let buildProgress = 0;

		function shatterChart() {
			const minP = Math.min(...candles.map(c => c.low));
			const maxP = Math.max(...candles.map(c => c.high));
			const range = maxP - minP;
			const cw = 12, gap = 4;
			const startX = w / 2 - (candles.length * (cw + gap)) / 2;
			candles.forEach((c, i) => {
				const x = startX + i * (cw + gap);
				const oY = h / 2 + 100 - ((c.open - minP) / range) * 300;
				const cY = h / 2 + 100 - ((c.close - minP) / range) * 300;
				const bodyH = Math.abs(cY - oY) || 2;
				const color = c.bull ? '#00ff88' : '#ff3333';
				for (let j = 0; j < 4; j++) {
					shards.push(new Shard(x + Math.random() * cw, Math.min(oY, cY) + Math.random() * bodyH, cw * 0.6, bodyH * 0.3, color, w, h));
				}
			});
		}

		function drawChartBuild() {
			if (phase === 'done') return;
			const minP = Math.min(...candles.map(c => c.low));
			const maxP = Math.max(...candles.map(c => c.high));
			const range = maxP - minP;
			const cw = 12, gap = 4;
			const startX = w / 2 - (candles.length * (cw + gap)) / 2;
			const builtCount = Math.floor(buildProgress * candles.length);
			for (let i = 0; i < builtCount && i < candles.length; i++) {
				const c = candles[i];
				const x = startX + i * (cw + gap);
				const oY = h / 2 + 100 - ((c.open - minP) / range) * 300;
				const cY = h / 2 + 100 - ((c.close - minP) / range) * 300;
				const hY = h / 2 + 100 - ((c.high - minP) / range) * 300;
				const lY = h / 2 + 100 - ((c.low - minP) / range) * 300;
				ctx.strokeStyle = c.bull ? 'rgba(0,255,136,0.4)' : 'rgba(255,51,51,0.4)';
				ctx.lineWidth = 1;
				ctx.beginPath(); ctx.moveTo(x + cw / 2, hY); ctx.lineTo(x + cw / 2, lY); ctx.stroke();
				ctx.fillStyle = c.bull ? '#00ff88' : '#ff3333';
				ctx.globalAlpha = 0.7;
				ctx.fillRect(x, Math.min(oY, cY), cw, Math.abs(cY - oY) || 2);
				ctx.globalAlpha = 1;
			}
		}

		function animate() {
			ctx.clearRect(0, 0, w, h);
			if (phase === 'build') drawChartBuild();
			if (phase === 'shatter') {
				shards = shards.filter(s => s.life > 0);
				shards.forEach(s => { s.update(); s.draw(ctx); });
				if (shards.length === 0) phase = 'done';
			}
			raf = requestAnimationFrame(animate);
		}
		animate();

		// Number rain
		for (let i = 0; i < 80; i++) {
			const num = document.createElement('div');
			num.className = 'sht-number-el' + (Math.random() > 0.5 ? ' sht-red' : '');
			Object.assign(num.style, {
				position: 'absolute', left: Math.random() * 100 + '%', top: '-20%',
				fontSize: (Math.random() * 0.5 + 0.5) + 'rem', opacity: '0',
				fontFamily: "'JetBrains Mono', monospace",
				color: Math.random() > 0.5 ? 'rgba(0,255,136,0.15)' : 'rgba(255,51,51,0.15)',
			});
			num.textContent = (Math.random() * 1000).toFixed(2);
			shtRain.appendChild(num);
		}

		const shtTl = gsap.timeline({
			scrollTrigger: { trigger: section, start: 'top 50%', toggleActions: 'play none none none' }
		});

		// Phase 1: Build chart
		shtTl.to({ val: 0 }, {
			val: 1, duration: 3, ease: 'power2.out',
			onUpdate: function () { buildProgress = (this.targets()[0] as { val: number }).val; }
		});

		// Phase 2: SHATTER
		shtTl.add(() => { phase = 'shatter'; shatterChart(); }, '+=0.3');
		shtTl.to(shtFlash, { opacity: 0.8, duration: 0.05 }, '<');
		shtTl.to(shtFlash, { opacity: 0, duration: 0.4, ease: 'power3.out' });

		shtTl.to(shtRain.querySelectorAll('.sht-number-el'), {
			opacity: () => Math.random() * 0.4 + 0.1,
			y: () => window.innerHeight * 1.2,
			duration: () => Math.random() * 2 + 1.5,
			stagger: { each: 0.02, from: 'random' }, ease: 'none'
		}, '<');

		// Phase 3: Content reveal
		shtTl.to(shtContent, { opacity: 1, duration: 0.5 }, '+=0.5');
		shtTl.to(section.querySelectorAll('.sht-word'), {
			opacity: 1, x: 0, filter: 'blur(0px)', duration: 0.8, stagger: 0.2, ease: 'power4.out'
		}, '<');
		shtTl.to(shtRule, { width: '200px', duration: 0.6, ease: 'power3.out' }, '-=0.3');
		shtTl.to(shtSub, { opacity: 1, duration: 0.6 }, '-=0.2');

		return () => { cancelAnimationFrame(raf); window.removeEventListener('resize', resize); };
	});
</script>

<section class="sht-section" bind:this={section}>
	<canvas class="sht-canvas" bind:this={canvas}></canvas>
	<div class="sht-number-rain" bind:this={shtRain}></div>
	<div class="sht-flash" bind:this={shtFlash}></div>
	<div class="sht-content" bind:this={shtContent}>
		<h1 class="sht-title">
			<span class="sht-word">EXPLOSIVE</span>
			<span class="sht-word">SWINGS</span>
		</h1>
		<div class="sht-rule" bind:this={shtRule}></div>
		<div class="sht-sub" bind:this={shtSub}>The setup. The signal. The execution.</div>
	</div>
</section>

<style>
	.sht-section { position: relative; height: 100vh; overflow: hidden; background: #060810; display: flex; align-items: center; justify-content: center; }
	.sht-canvas { position: absolute; inset: 0; z-index: 1; width: 100%; height: 100%; }
	.sht-number-rain { position: absolute; inset: 0; z-index: 2; pointer-events: none; overflow: hidden; }
	.sht-flash { position: absolute; inset: 0; background: #fff; opacity: 0; z-index: 8; pointer-events: none; }
	.sht-content { position: relative; z-index: 10; text-align: center; opacity: 0; }
	.sht-title { font-family: 'Bebas Neue', sans-serif; font-size: clamp(4rem, 14vw, 12rem); line-height: 0.85; color: #fff; letter-spacing: 0.03em; }
	.sht-word { display: block; opacity: 0; transform: translateX(-100px); filter: blur(10px); }
	.sht-word:nth-child(2) { color: #00ff88; }
	.sht-rule { width: 0; height: 2px; margin: 20px auto; background: linear-gradient(90deg, #00ff88, #00aaff); }
	.sht-sub { font-family: 'JetBrains Mono', monospace; font-size: 0.7rem; letter-spacing: 0.4em; text-transform: uppercase; color: rgba(255,255,255,0.3); opacity: 0; }
</style>
