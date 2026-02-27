<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	let section: HTMLElement;
	let canvas: HTMLCanvasElement;
	let crackSvg: SVGSVGElement;
	let impTitle: HTMLElement;
	let impSubtitle: HTMLElement;
	let impFlash: HTMLElement;
	let impDustCloud: HTMLElement;
	let impGroundLine: HTMLElement;

	onMount(() => {
		gsap.registerPlugin(ScrollTrigger);
		const ctx = canvas.getContext('2d')!;
		let w = (canvas.width = window.innerWidth);
		let h = (canvas.height = window.innerHeight);
		let raf: number;

		function resize() { w = canvas.width = window.innerWidth; h = canvas.height = window.innerHeight; }
		window.addEventListener('resize', resize);

		type DustParticle = { x: number; y: number; vx: number; vy: number; size: number; life: number; decay: number; gray: number };
		let dustParticles: DustParticle[] = [];

		function spawnDust(cx: number, cy: number, count: number) {
			for (let i = 0; i < count; i++) {
				const angle = Math.random() * Math.PI * 2;
				const speed = Math.random() * 8 + 2;
				dustParticles.push({
					x: cx + (Math.random() - 0.5) * 100, y: cy + (Math.random() - 0.5) * 50,
					vx: Math.cos(angle) * speed, vy: Math.sin(angle) * speed * 0.5 - Math.random() * 3,
					size: Math.random() * 40 + 10, life: 1,
					decay: 0.005 + Math.random() * 0.008,
					gray: Math.floor(Math.random() * 100 + 100),
				});
			}
		}

		function animateDust() {
			ctx.clearRect(0, 0, w, h);
			dustParticles = dustParticles.filter(p => p.life > 0);
			dustParticles.forEach(p => {
				p.x += p.vx; p.y += p.vy; p.vx *= 0.98; p.vy *= 0.98; p.vy += 0.02;
				p.life -= p.decay; p.size *= 1.003;
				ctx.globalAlpha = p.life * 0.2;
				ctx.beginPath(); ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
				ctx.fillStyle = `rgb(${p.gray},${p.gray - 20},${p.gray - 40})`; ctx.fill();
			});
			ctx.globalAlpha = 1;
			raf = requestAnimationFrame(animateDust);
		}
		animateDust();

		// Generate crack paths
		function generateCracks(cx: number, cy: number) {
			const paths: SVGPathElement[] = [];
			for (let i = 0; i < 12; i++) {
				let x = cx, y = cy;
				let angle = (Math.PI * 2 / 12) * i + (Math.random() - 0.5) * 0.5;
				let d = `M${x},${y}`;
				const segments = Math.floor(Math.random() * 8 + 4);
				for (let j = 0; j < segments; j++) {
					const len = Math.random() * 80 + 30;
					angle += (Math.random() - 0.5) * 0.8;
					x += Math.cos(angle) * len; y += Math.sin(angle) * len;
					d += ` L${x},${y}`;
					if (Math.random() > 0.6) {
						let bx = x, by = y;
						const bAngle = angle + (Math.random() - 0.5) * 1.2;
						const bLen = Math.random() * 50 + 20;
						bx += Math.cos(bAngle) * bLen; by += Math.sin(bAngle) * bLen;
						const branch = document.createElementNS('http://www.w3.org/2000/svg', 'path');
						branch.setAttribute('d', `M${x},${y} L${bx},${by}`);
						branch.setAttribute('fill', 'none');
						branch.setAttribute('stroke', 'rgba(255,200,150,0.3)');
						branch.setAttribute('stroke-width', '1');
						branch.style.strokeDasharray = '1000';
						branch.style.strokeDashoffset = '1000';
						crackSvg.appendChild(branch);
						paths.push(branch);
					}
				}
				const path = document.createElementNS('http://www.w3.org/2000/svg', 'path');
				path.setAttribute('d', d);
				path.setAttribute('fill', 'none');
				path.setAttribute('stroke', 'rgba(255,180,120,0.4)');
				path.setAttribute('stroke-width', '1.5');
				path.style.strokeDasharray = '2000';
				path.style.strokeDashoffset = '2000';
				crackSvg.appendChild(path);
				paths.push(path);
			}
			return paths;
		}

		// Debris elements
		for (let i = 0; i < 30; i++) {
			const d = document.createElement('div');
			d.className = 'imp-debris-el';
			Object.assign(d.style, {
				position: 'absolute', left: '50%', top: '55%',
				width: Math.random() * 12 + 3 + 'px', height: Math.random() * 8 + 2 + 'px',
				background: '#8a7560', borderRadius: Math.random() > 0.5 ? '1px' : '0',
				zIndex: '6', opacity: '0', pointerEvents: 'none',
			});
			section.appendChild(d);
		}

		const crackPaths = generateCracks(960, 540);

		function screenShake(target: Element | string, intensity = 40, duration = 0.8) {
			const tl = gsap.timeline();
			const steps = Math.floor(duration / 0.03);
			for (let i = 0; i < steps; i++) {
				const decay = 1 - i / steps;
				tl.set(target, { x: (Math.random() - 0.5) * intensity * decay, y: (Math.random() - 0.5) * intensity * decay }, i * 0.03);
			}
			tl.set(target, { x: 0, y: 0 }); return tl;
		}

		const impTl = gsap.timeline({
			scrollTrigger: { trigger: section, start: 'top 60%', toggleActions: 'play none none none' }
		});

		impTl.to(impTitle, { opacity: 1, y: 0, scale: 1, duration: 0.15, ease: 'power4.in' });
		impTl.to(impFlash, { opacity: 1, duration: 0.05 }, '<');
		impTl.to(impFlash, { opacity: 0, duration: 0.6, ease: 'power3.out' });
		impTl.add(screenShake(section.querySelector('.imp-content')!, 40, 0.8), '<');

		// Shockwave rings
		section.querySelectorAll('.imp-shockring').forEach((ring, i) => {
			impTl.to(ring, { width: '200vmax', height: '200vmax', opacity: 0.6, duration: 0.01 }, `<+=${i * 0.15}`);
			impTl.to(ring, { width: '300vmax', height: '300vmax', opacity: 0, duration: 1.2, ease: 'power2.out' });
		});

		impTl.to(impDustCloud, { width: '120vw', opacity: 0.7, duration: 0.8, ease: 'power2.out' }, '<');
		impTl.to(impDustCloud, { opacity: 0, duration: 2 }, '+=0.5');
		impTl.add(() => spawnDust(w / 2, h * 0.55, 200), '<-0.8');

		impTl.to(crackPaths, { strokeDashoffset: 0, duration: 0.6, stagger: 0.03, ease: 'power2.out' }, '<');
		impTl.to(impGroundLine, { scaleX: 1, duration: 0.3, ease: 'power4.out' }, '<');

		impTl.to(section.querySelectorAll('.imp-debris-el'), {
			opacity: 1,
			x: () => (Math.random() - 0.5) * window.innerWidth * 0.8,
			y: () => (Math.random() - 0.5) * window.innerHeight * 0.6,
			rotation: () => Math.random() * 720 - 360,
			duration: 1, ease: 'power2.out', stagger: { each: 0.02, from: 'random' }
		}, '<');
		impTl.to(section.querySelectorAll('.imp-debris-el'), { opacity: 0, duration: 0.5 }, '+=0.3');
		impTl.to(impTitle, { textShadow: '0 5px 30px rgba(0,0,0,0.5)', duration: 1, ease: 'elastic.out(1.2,0.4)' }, '<-1');
		impTl.to(impSubtitle, { opacity: 1, y: 0, duration: 1, ease: 'power3.out' }, '-=0.5');

		return () => { cancelAnimationFrame(raf); window.removeEventListener('resize', resize); };
	});
</script>

<section class="imp-section" bind:this={section}>
	<canvas class="imp-dust-canvas" bind:this={canvas}></canvas>
	<svg class="imp-crack-svg" bind:this={crackSvg} viewBox="0 0 1920 1080" preserveAspectRatio="xMidYMid slice"></svg>
	<div class="imp-ground"><div class="imp-ground-line" bind:this={impGroundLine}></div></div>
	<div class="imp-flash" bind:this={impFlash}></div>
	<div class="imp-dust-cloud" bind:this={impDustCloud}></div>
	<div class="imp-shockring"></div>
	<div class="imp-shockring" style="border-color:rgba(255,150,50,0.3)"></div>
	<div class="imp-content">
		<h1 class="imp-title" bind:this={impTitle}>IMPACT</h1>
		<p class="imp-subtitle" bind:this={impSubtitle}>When preparation meets opportunity</p>
	</div>
</section>

<style>
	.imp-section { position: relative; height: 100vh; overflow: hidden; background: #0a0908; display: flex; align-items: center; justify-content: center; }
	.imp-dust-canvas { position: absolute; inset: 0; z-index: 3; width: 100%; height: 100%; }
	.imp-crack-svg { position: absolute; inset: 0; z-index: 4; pointer-events: none; }
	.imp-ground { position: absolute; bottom: 0; left: 0; right: 0; height: 40%; z-index: 1; background: linear-gradient(0deg, #1a1612 0%, #0a0908 100%); }
	.imp-ground-line { position: absolute; top: 0; left: 0; right: 0; height: 1px; background: linear-gradient(90deg, transparent 10%, rgba(255,180,80,0.2) 50%, transparent 90%); transform: scaleX(0); }
	.imp-flash { position: absolute; inset: 0; z-index: 7; pointer-events: none; background: radial-gradient(circle at 50% 55%, rgba(255,200,100,0.8), transparent 60%); opacity: 0; }
	.imp-dust-cloud { position: absolute; bottom: 25%; left: 50%; transform: translateX(-50%); width: 0; height: 200px; z-index: 2; pointer-events: none; background: radial-gradient(ellipse at 50% 100%, rgba(180,150,120,0.4), transparent 70%); filter: blur(30px); opacity: 0; }
	.imp-shockring { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 10px; height: 10px; border-radius: 50%; border: 2px solid rgba(255,180,80,0.6); opacity: 0; z-index: 5; pointer-events: none; }
	.imp-content { position: relative; z-index: 10; text-align: center; }
	.imp-title { font-family: 'Vast Shadow', serif; font-size: clamp(4rem, 13vw, 11rem); color: #fff; line-height: 0.9; opacity: 0; transform: translateY(-200vh) scale(1.5); text-shadow: 0 20px 60px rgba(0,0,0,0.8); }
	.imp-subtitle { font-family: 'Oswald', sans-serif; font-weight: 300; font-size: clamp(0.9rem, 2vw, 1.4rem); letter-spacing: 0.4em; text-transform: uppercase; color: rgba(255,180,80,0.5); margin-top: 30px; opacity: 0; transform: translateY(40px); }
</style>
