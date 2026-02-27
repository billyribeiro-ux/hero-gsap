<script context="module" lang="ts">
	type PType = 'spark' | 'ember' | 'smoke' | 'debris';

	class Particle {
		x: number; y: number; vx: number; vy: number;
		life: number; decay: number; size: number;
		rotation: number; rotSpeed: number;
		gravity: number; drag: number;
		r: number; g: number; b: number;
		type: PType;

		constructor(x: number, y: number, type: PType, w: number, h: number) {
			void h;
			this.x = x; this.y = y; this.type = type;
			const angle = Math.random() * Math.PI * 2;
			const speed = type === 'spark' ? Math.random() * 18 + 8 :
				type === 'debris' ? Math.random() * 12 + 4 :
				type === 'smoke' ? Math.random() * 3 + 0.5 : Math.random() * 6 + 2;
			this.vx = Math.cos(angle) * speed;
			this.vy = Math.sin(angle) * speed - (type === 'smoke' ? 2 : 0);
			this.life = 1;
			this.decay = type === 'spark' ? 0.02 + Math.random() * 0.03 :
				type === 'smoke' ? 0.003 + Math.random() * 0.005 : 0.008 + Math.random() * 0.015;
			this.size = type === 'spark' ? Math.random() * 3 + 1 :
				type === 'smoke' ? Math.random() * 60 + 30 :
				type === 'debris' ? Math.random() * 5 + 2 : Math.random() * 4 + 2;
			this.rotation = Math.random() * Math.PI * 2;
			this.rotSpeed = (Math.random() - 0.5) * 0.2;
			this.gravity = type === 'smoke' ? -0.02 : type === 'spark' ? 0.15 : 0.1;
			this.drag = type === 'smoke' ? 0.99 : 0.97;
			this.r = type === 'spark' ? 255 : type === 'debris' ? 180 + Math.random() * 50 : 200;
			this.g = type === 'spark' ? 100 + Math.random() * 100 : type === 'debris' ? 120 + Math.random() * 40 : 150;
			this.b = type === 'spark' ? 0 : type === 'debris' ? 60 : 100;
		}
		update() {
			this.vx *= this.drag; this.vy *= this.drag;
			this.vy += this.gravity;
			this.x += this.vx; this.y += this.vy;
			this.rotation += this.rotSpeed;
			this.life -= this.decay;
		}
		draw(c: CanvasRenderingContext2D) {
			if (this.life <= 0) return;
			c.save(); c.globalAlpha = this.life;
			if (this.type === 'spark') {
				const len = Math.sqrt(this.vx * this.vx + this.vy * this.vy) * 3;
				const angle = Math.atan2(this.vy, this.vx);
				c.beginPath();
				c.moveTo(this.x - Math.cos(angle) * len, this.y - Math.sin(angle) * len);
				c.lineTo(this.x, this.y);
				c.strokeStyle = `rgba(${this.r},${this.g},${this.b},${this.life})`;
				c.lineWidth = this.size * 0.6; c.stroke();
				c.beginPath(); c.arc(this.x, this.y, this.size * 0.5, 0, Math.PI * 2);
				c.fillStyle = `rgba(255,255,200,${this.life * 0.8})`; c.fill();
			} else if (this.type === 'smoke') {
				c.globalAlpha = this.life * 0.15;
				c.beginPath(); c.arc(this.x, this.y, this.size, 0, Math.PI * 2);
				c.fillStyle = `rgba(${this.r},${this.g},${this.b},1)`; c.fill();
			} else if (this.type === 'debris') {
				c.translate(this.x, this.y); c.rotate(this.rotation);
				c.fillStyle = `rgba(${this.r},${this.g},${this.b},${this.life})`;
				c.fillRect(-this.size / 2, -this.size / 2, this.size, this.size * 0.6);
			} else {
				c.beginPath(); c.arc(this.x, this.y, this.size, 0, Math.PI * 2);
				c.fillStyle = `rgba(${this.r},${this.g},${this.b},${this.life})`; c.fill();
				c.shadowBlur = 15; c.shadowColor = `rgba(255,100,0,${this.life * 0.5})`; c.fill();
			}
			c.restore();
		}
	}
</script>

<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';

	let section: HTMLElement;
	let canvas: HTMLCanvasElement;
	let detFlash: HTMLElement;
	let detShockwave: HTMLElement;
	let detShockwave2: HTMLElement;
	let detHeatwave: HTMLElement;
	let detContent: HTMLElement;
	let detPre: HTMLElement;
	let detTitle: HTMLElement;
	let detSub: HTMLElement;
	let detVignette: HTMLElement;

	onMount(() => {
		const ctx = canvas.getContext('2d')!;
		let w = (canvas.width = window.innerWidth);
		let h = (canvas.height = window.innerHeight);
		let raf: number;

		function resize() {
			w = canvas.width = window.innerWidth;
			h = canvas.height = window.innerHeight;
		}
		window.addEventListener('resize', resize);


		let particles: Particle[] = [];
		let detonated = false;

		function detonate() {
			if (detonated) return;
			detonated = true;
			const cx = w / 2, cy = h / 2;
			for (let i = 0; i < 300; i++) particles.push(new Particle(cx, cy, 'spark', w, h));
			for (let i = 0; i < 80; i++) particles.push(new Particle(cx, cy, 'ember', w, h));
			for (let i = 0; i < 40; i++) particles.push(new Particle(cx, cy, 'smoke', w, h));
			for (let i = 0; i < 60; i++) particles.push(new Particle(cx, cy, 'debris', w, h));
			setTimeout(() => {
				for (let i = 0; i < 150; i++) particles.push(new Particle(cx + (Math.random() - 0.5) * 100, cy + (Math.random() - 0.5) * 100, 'spark', w, h));
				for (let i = 0; i < 30; i++) particles.push(new Particle(cx, cy, 'smoke', w, h));
			}, 200);
			setTimeout(() => {
				for (let i = 0; i < 80; i++) particles.push(new Particle(cx + (Math.random() - 0.5) * 200, cy + (Math.random() - 0.5) * 200, 'ember', w, h));
			}, 500);
		}

		// Pre-explosion ambient
		const preParts: { x: number; y: number; size: number; speed: number; opacity: number }[] = [];
		for (let i = 0; i < 50; i++) preParts.push({
			x: Math.random() * w, y: Math.random() * h,
			size: Math.random() * 2 + 0.5, speed: Math.random() * 0.5 + 0.2, opacity: Math.random() * 0.3,
		});

		function animate() {
			ctx.clearRect(0, 0, w, h);
			if (!detonated) {
				preParts.forEach(p => {
					p.y -= p.speed;
					if (p.y < 0) { p.y = h; p.x = Math.random() * w; }
					ctx.beginPath(); ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
					ctx.fillStyle = `rgba(255,80,0,${p.opacity})`; ctx.fill();
				});
			}
			particles = particles.filter(p => p.life > 0);
			particles.forEach(p => { p.update(); p.draw(ctx); });
			raf = requestAnimationFrame(animate);
		}
		animate();

		// Split title into chars
		const text = 'DETONATE';
		detTitle.innerHTML = text.split('').map(c => `<span class="det-char">${c}</span>`).join('');

		// Floating embers
		for (let i = 0; i < 25; i++) {
			const ember = document.createElement('div');
			ember.className = 'det-ember-el';
			ember.style.cssText = `position:absolute;width:3px;height:3px;border-radius:50%;background:#ff6a00;z-index:5;opacity:0;pointer-events:none;box-shadow:0 0 6px #ff6a00;left:${Math.random() * 100}%;top:${Math.random() * 100}%`;
			section.appendChild(ember);
		}

		function screenShake(target: string | Element, intensity = 20, duration = 0.6) {
			const tl = gsap.timeline();
			const steps = Math.floor(duration / 0.03);
			for (let i = 0; i < steps; i++) {
				const d = 1 - i / steps;
				tl.set(target, { x: (Math.random() - 0.5) * intensity * d, y: (Math.random() - 0.5) * intensity * d }, i * 0.03);
			}
			tl.set(target, { x: 0, y: 0 });
			return tl;
		}

		const master = gsap.timeline({ delay: 0.8 });
		master.to(detPre, { opacity: 1, duration: 0.8, ease: 'power2.out' });
		master.to(detPre, { textShadow: '0 0 40px rgba(255,60,0,1)', scale: 1.05, duration: 1.5, ease: 'power2.in' }, '+=0.5');
		master.add(() => detonate(), '+=0.3');

		master.to(detFlash, { opacity: 1, duration: 0.05 }, '<');
		master.to(detFlash, { opacity: 0, duration: 0.8, ease: 'power4.out' }, '<+=0.05');
		master.add(screenShake(detContent, 30, 0.8), '<');
		master.add(screenShake(detVignette, 15, 0.6), '<');

		master.to(detShockwave, { width: '150vmax', height: '150vmax', opacity: 0.8, duration: 0.01 }, '<');
		master.to(detShockwave, { width: '250vmax', height: '250vmax', opacity: 0, borderWidth: '1px', duration: 1.2, ease: 'power2.out' });
		master.to(detShockwave2, { width: '200vmax', height: '200vmax', opacity: 0.5, duration: 0.01 }, '<-0.8');
		master.to(detShockwave2, { width: '300vmax', height: '300vmax', opacity: 0, duration: 1.5, ease: 'power2.out' });

		master.to(detHeatwave, { opacity: 0.6, duration: 0.3 }, '<-1');
		master.to(detHeatwave, { opacity: 0, duration: 3 });

		master.to(section.querySelectorAll('.det-char'), {
			opacity: 1, filter: 'blur(0px)', scale: 1, y: 0,
			duration: 1.2, stagger: { each: 0.06, from: 'center' }, ease: 'power4.out'
		}, '<-0.5');
		master.to(detSub, { opacity: 1, y: 0, duration: 1, ease: 'power3.out' }, '-=0.4');

		master.to(section.querySelectorAll('.det-ember-el'), {
			opacity: () => Math.random() * 0.8 + 0.2,
			y: () => -(Math.random() * 300 + 100),
			x: () => (Math.random() - 0.5) * 200,
			duration: () => Math.random() * 3 + 2,
			stagger: { each: 0.08, from: 'random' }, ease: 'power1.out', repeat: -1, repeatDelay: 0.5
		}, '<');
		master.to(detPre, { opacity: 0.4, duration: 2 }, '<');

		return () => { cancelAnimationFrame(raf); window.removeEventListener('resize', resize); };
	});
</script>

<!-- SVG filters -->
<svg style="position:absolute;width:0;height:0">
	<defs>
		<filter id="det-chromatic">
			<feOffset in="SourceGraphic" dx="3" dy="0" result="red"/>
			<feOffset in="SourceGraphic" dx="-3" dy="0" result="blue"/>
			<feColorMatrix in="red" type="matrix" values="1 0 0 0 0  0 0 0 0 0  0 0 0 0 0  0 0 0 1 0" result="redOnly"/>
			<feColorMatrix in="blue" type="matrix" values="0 0 0 0 0  0 0 0 0 0  0 0 1 0 0  0 0 0 1 0" result="blueOnly"/>
			<feColorMatrix in="SourceGraphic" type="matrix" values="0 0 0 0 0  0 1 0 0 0  0 0 0 0 0  0 0 0 1 0" result="greenOnly"/>
			<feBlend in="redOnly" in2="greenOnly" mode="screen" result="rg"/>
			<feBlend in="rg" in2="blueOnly" mode="screen"/>
		</filter>
		<filter id="det-heat">
			<feTurbulence type="turbulence" baseFrequency="0.02 0.05" numOctaves="2" seed="2" result="turbulence">
				<animate attributeName="baseFrequency" dur="3s" values="0.02 0.05;0.03 0.07;0.02 0.05" repeatCount="indefinite"/>
			</feTurbulence>
			<feDisplacementMap in="SourceGraphic" in2="turbulence" scale="8" xChannelSelector="R" yChannelSelector="G"/>
		</filter>
	</defs>
</svg>

<section class="det-section" bind:this={section}>
	<canvas class="det-canvas" bind:this={canvas}></canvas>
	<div class="det-flash" bind:this={detFlash}></div>
	<div class="det-shockwave" bind:this={detShockwave}></div>
	<div class="det-shockwave2" bind:this={detShockwave2}></div>
	<div class="det-heatwave" bind:this={detHeatwave}></div>
	<div class="det-content" style="filter:url(#det-chromatic)" bind:this={detContent}>
		<div class="det-pre" bind:this={detPre}>[ SYSTEM CRITICAL ]</div>
		<h1 class="det-title" aria-label="DETONATE" bind:this={detTitle}></h1>
		<p class="det-sub" bind:this={detSub}>THE MOVE PRIOR TO THE MOVE</p>
	</div>
	<div class="det-vignette" bind:this={detVignette}></div>
	<div class="det-grain"></div>
</section>

<style>
	.det-section {
		position: relative; height: 100vh; overflow: hidden;
		background: #000; display: flex; align-items: center; justify-content: center;
	}
	.det-canvas { position: absolute; inset: 0; z-index: 1; width: 100%; height: 100%; }
	.det-flash { position: absolute; inset: 0; background: #fff; opacity: 0; z-index: 2; pointer-events: none; mix-blend-mode: overlay; }
	.det-shockwave {
		position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
		width: 10px; height: 10px; border-radius: 50%;
		border: 3px solid rgba(255,120,0,0.8); opacity: 0; z-index: 3;
		box-shadow: 0 0 40px rgba(255,80,0,0.4), inset 0 0 20px rgba(255,60,0,0.2); pointer-events: none;
	}
	.det-shockwave2 {
		position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
		width: 10px; height: 10px; border-radius: 50%;
		border: 2px solid rgba(255,200,50,0.5); opacity: 0; z-index: 3; pointer-events: none;
	}
	.det-heatwave { position: absolute; inset: 0; z-index: 3; pointer-events: none; filter: url(#det-heat); opacity: 0; }
	.det-content { position: relative; z-index: 10; text-align: center; }
	.det-pre {
		font-family: 'JetBrains Mono', monospace; font-size: 0.6rem;
		letter-spacing: 0.6em; text-transform: uppercase;
		color: #ff6a00; margin-bottom: 20px; opacity: 0;
		text-shadow: 0 0 20px rgba(255,106,0,0.5);
	}
	.det-title {
		font-family: 'Anton', sans-serif; font-size: clamp(5rem, 16vw, 14rem);
		line-height: 0.85; letter-spacing: 0.02em; color: #fff;
	}
	:global(.det-char) {
		display: inline-block; opacity: 0;
		filter: blur(20px); transform: scale(3) translateY(40px);
		text-shadow: 0 0 80px rgba(255,100,0,0.8);
	}
	.det-sub {
		font-family: 'Oswald', sans-serif; font-size: clamp(1rem, 2.5vw, 1.8rem);
		font-weight: 300; letter-spacing: 0.3em; text-transform: uppercase;
		color: rgba(255,255,255,0.4); margin-top: 30px; opacity: 0; transform: translateY(30px);
	}
	.det-vignette {
		position: absolute; inset: 0; z-index: 4; pointer-events: none;
		background: radial-gradient(ellipse at center, transparent 40%, rgba(0,0,0,0.7) 100%);
	}
	.det-grain {
		position: absolute; inset: 0; z-index: 6; pointer-events: none; opacity: 0.08;
		background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 512 512' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.7' numOctaves='5' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
		background-size: 256px;
	}
</style>
