<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	let section: HTMLElement;
	let trBlackout: HTMLElement;
	let trWhip: HTMLElement;
	let trLensFlare: HTMLElement;
	let trLensStreak: HTMLElement;
	let trCounter: HTMLElement;
	let trFinalTitle: HTMLElement;
	let trFinalTag: HTMLElement;
	let trFinalCta: HTMLButtonElement;
	let trFilmTop: HTMLElement;
	let trFilmBottom: HTMLElement;
	let trScene1: HTMLElement;
	let trScene2: HTMLElement;
	let trScene3: HTMLElement;

	onMount(() => {
		gsap.registerPlugin(ScrollTrigger);

		function screenShake(target: Element | string, intensity = 25, duration = 0.6) {
			const tl = gsap.timeline();
			const steps = Math.floor(duration / 0.03);
			for (let i = 0; i < steps; i++) {
				const d = 1 - i / steps;
				tl.set(target, { x: (Math.random() - 0.5) * intensity * d, y: (Math.random() - 0.5) * intensity * d }, i * 0.03);
			}
			tl.set(target, { x: 0, y: 0 }); return tl;
		}

		const trTl = gsap.timeline({
			scrollTrigger: { trigger: section, start: 'top 50%', toggleActions: 'play none none none' }
		});

		// Film bars
		trTl.to(trFilmTop, { height: '12%', duration: 0.8, ease: 'power3.out' }, 0);
		trTl.to(trFilmBottom, { height: '12%', duration: 0.8, ease: 'power3.out' }, 0);

		// Scene 1
		trTl.to(trBlackout, { opacity: 0, duration: 0.8 }, 0.5);
		trTl.to(trScene1, { opacity: 1, duration: 0.5 }, 1);
		trTl.fromTo(trScene1, { scale: 1 }, { scale: 1.05, duration: 2.5, ease: 'none' }, 1);

		// Blackout cut
		trTl.to(trBlackout, { opacity: 1, duration: 0.08 }, 3.5);
		trTl.set(trScene1, { opacity: 0 }, 3.55);

		// Scene 2
		trTl.to(trBlackout, { opacity: 0, duration: 0.08 }, 4);
		trTl.to(trScene2, { opacity: 1, duration: 0.05 }, 4);
		trTl.fromTo(trScene2, { scale: 1.1 }, { scale: 1, duration: 2, ease: 'power2.out' }, 4);

		// Whip transition
		trTl.to(trWhip, { opacity: 1, duration: 0.08 }, 6);
		trTl.to(trWhip, { opacity: 0, duration: 0.15 }, 6.12);
		trTl.set(trScene2, { opacity: 0 }, 6.1);

		// Scene 3 — fast cut
		trTl.to(trScene3, { opacity: 1, duration: 0.05 }, 6.3);
		trTl.add(screenShake(trScene3, 10, 0.3), 6.3);

		// Blackout to countdown
		trTl.to([trScene3], { opacity: 0, duration: 0.08 }, 8);
		trTl.to(trBlackout, { opacity: 1, duration: 0.05 }, 8);

		// Countdown: 3 2 1
		['3', '2', '1'].forEach((num, i) => {
			const t = 8.5 + i * 0.8;
			trTl.set(trCounter, { textContent: num, opacity: 0, scale: 2 }, t);
			trTl.to(trCounter, { opacity: 1, scale: 1, duration: 0.15, ease: 'power4.out' }, t);
			trTl.to(trCounter, { opacity: 0, duration: 0.2 }, t + 0.5);
			trTl.to(trBlackout, { opacity: 0.5, duration: 0.05 }, t);
			trTl.to(trBlackout, { opacity: 1, duration: 0.1 }, t + 0.15);
		});

		// FINAL SLAM
		const slamTime = 11;
		trTl.to(trBlackout, { opacity: 0, duration: 0.05 }, slamTime);
		trTl.to(trFinalTitle, { opacity: 1, scale: 1, filter: 'blur(0px)', duration: 0.4, ease: 'power4.out' }, slamTime);
		trTl.add(screenShake(section.querySelector('.tr-final')!, 25, 0.6), slamTime);
		trTl.to(trBlackout, { opacity: 0.8, duration: 0.03 }, slamTime);
		trTl.to(trBlackout, { opacity: 0, duration: 0.5, ease: 'power3.out' }, slamTime + 0.03);

		trTl.to(trLensFlare, { opacity: 1, width: '600px', height: '600px', duration: 0.3, ease: 'power2.out' }, slamTime + 0.1);
		trTl.to(trLensFlare, { opacity: 0, width: '100px', height: '100px', duration: 1.5, ease: 'power2.in' }, slamTime + 0.5);
		trTl.to(trLensStreak, { opacity: 1, duration: 0.1 }, slamTime + 0.15);
		trTl.to(trLensStreak, { opacity: 0, duration: 1 }, slamTime + 0.4);
		trTl.to(trFinalTag, { opacity: 1, duration: 0.8, ease: 'power2.out' }, slamTime + 0.8);
		trTl.to(trFinalCta, { opacity: 1, duration: 0.8, ease: 'power2.out' }, slamTime + 1.2);
	});
</script>

<section class="tr-section" bind:this={section}>
	<div class="tr-grain"></div>
	<div class="tr-film-bars tr-top" bind:this={trFilmTop}></div>
	<div class="tr-film-bars tr-bottom" bind:this={trFilmBottom}></div>
	<div class="tr-blackout" bind:this={trBlackout}></div>
	<div class="tr-whip" bind:this={trWhip}></div>
	<div class="tr-lens-flare" bind:this={trLensFlare}></div>
	<div class="tr-lens-streak" bind:this={trLensStreak}></div>

	<div class="tr-scene tr-scene-1" bind:this={trScene1}>
		<div class="tr-scene-text">They said the market<br>was <em>unpredictable</em></div>
	</div>
	<div class="tr-scene tr-scene-2" bind:this={trScene2}>
		<div class="tr-scene-text">They said no one could<br>see it <em>coming</em></div>
	</div>
	<div class="tr-scene tr-scene-3" bind:this={trScene3}>
		<div class="tr-scene-text"><em>They</em> were wrong</div>
	</div>

	<div class="tr-counter" bind:this={trCounter}></div>

	<div class="tr-final">
		<h1 class="tr-final-title" bind:this={trFinalTitle}>RTP</h1>
		<div class="tr-final-tag" bind:this={trFinalTag}>Revolution Trading Pros</div>
		<button class="tr-final-cta" bind:this={trFinalCta}>ENTER THE ARENA</button>
	</div>
</section>

<style>
	.tr-section { position: relative; height: 100vh; overflow: hidden; background: #000; }

	.tr-grain { position: absolute; inset: 0; z-index: 40; pointer-events: none; opacity: 0.05; background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 512 512' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.8' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E"); background-size: 256px; mix-blend-mode: overlay; }
	.tr-film-bars { position: absolute; z-index: 45; pointer-events: none; left: 0; right: 0; background: #000; }
	.tr-top    { top: 0; height: 0; }
	.tr-bottom { bottom: 0; height: 0; }
	.tr-blackout { position: absolute; inset: 0; background: #000; z-index: 50; }
	.tr-whip { position: absolute; inset: 0; z-index: 20; pointer-events: none; background: linear-gradient(90deg, #000 0%, transparent 15%, transparent 85%, #000 100%); opacity: 0; }
	.tr-lens-flare { position: absolute; top: 50%; left: 50%; transform: translate(-50%,-50%); width: 300px; height: 300px; z-index: 15; pointer-events: none; opacity: 0; background: radial-gradient(circle, rgba(255,200,100,0.6) 0%, rgba(255,150,50,0.2) 30%, transparent 70%); mix-blend-mode: screen; }
	.tr-lens-streak { position: absolute; top: 50%; left: 0; right: 0; height: 2px; z-index: 15; background: linear-gradient(90deg, transparent, rgba(255,200,150,0.4), transparent); pointer-events: none; opacity: 0; transform: translateY(-50%); }

	.tr-scene { position: absolute; inset: 0; display: flex; align-items: center; justify-content: center; opacity: 0; z-index: 5; }
	.tr-scene-text { font-family: 'Oswald', sans-serif; font-weight: 300; font-size: clamp(1.5rem, 4vw, 3rem); letter-spacing: 0.2em; text-transform: uppercase; color: rgba(255,255,255,0.8); text-align: center; line-height: 1.5; }
	.tr-scene-text :global(em) { font-family: 'Instrument Serif', serif; font-style: italic; font-size: 1.3em; color: #ff3c00; letter-spacing: 0; }

	.tr-counter { position: absolute; top: 50%; left: 50%; transform: translate(-50%,-50%); font-family: 'Orbitron', sans-serif; font-size: clamp(6rem, 20vw, 15rem); font-weight: 900; color: #fff; z-index: 30; opacity: 0; text-shadow: 0 0 60px rgba(255,60,0,0.5); }

	.tr-final { position: absolute; inset: 0; display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 10; }
	.tr-final-title { font-family: 'Anton', sans-serif; font-size: clamp(5rem, 18vw, 16rem); line-height: 0.8; color: #fff; opacity: 0; transform: scale(5); filter: blur(30px); }
	.tr-final-tag { font-family: 'JetBrains Mono', monospace; font-size: 0.7rem; letter-spacing: 0.5em; text-transform: uppercase; color: #ff3c00; margin-top: 30px; opacity: 0; }
	.tr-final-cta { margin-top: 50px; padding: 16px 50px; border: 1px solid rgba(255,255,255,0.2); background: transparent; color: #fff; font-family: 'Oswald', sans-serif; font-size: 0.9rem; letter-spacing: 0.3em; text-transform: uppercase; cursor: pointer; opacity: 0; transition: all 0.3s; position: relative; overflow: hidden; }
	.tr-final-cta:hover { border-color: #ff3c00; color: #ff3c00; }
	.tr-final-cta::after { content: ''; position: absolute; top: 0; left: -100%; width: 100%; height: 100%; background: linear-gradient(90deg, transparent, rgba(255,60,0,0.1), transparent); transition: left 0.5s; }
	.tr-final-cta:hover::after { left: 100%; }
</style>
