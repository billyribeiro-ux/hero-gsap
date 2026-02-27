<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { Warning, Eye } from 'phosphor-svelte';

	let section: HTMLElement;
	let titleRef: HTMLElement;
	let titleRedRef: HTMLElement;
	let titleBlueRef: HTMLElement;
	let subtitleRef: HTMLElement;
	let ctaRef: HTMLElement;
	let noiseCanvas: HTMLCanvasElement;
	let scanlineRef: HTMLElement;
	let glitchBarsRef: HTMLElement;
	let statusRef: HTMLElement;

	const GLITCH_CHARS = '!@#$%^&*<>?/|\\{}[]0123456789ABCDEF';

	function randomGlitchChar() {
		return GLITCH_CHARS[Math.floor(Math.random() * GLITCH_CHARS.length)];
	}

	function scrambleText(el: HTMLElement, finalText: string, duration = 1.2) {
		const chars = finalText.split('');
		let iteration = 0;
		const total = chars.length * 6;
		const iv = setInterval(() => {
			el.textContent = chars
				.map((c, i) => {
					if (c === ' ' || c === '\n') return c;
					if (i < Math.floor(iteration / 6)) return c;
					return randomGlitchChar();
				})
				.join('');
			iteration++;
			if (iteration >= total) {
				el.textContent = finalText;
				clearInterval(iv);
			}
		}, (duration * 1000) / total);
	}

	function triggerGlitch() {
		const shifts = [
			{ x: -4, y: 0, scaleX: 1.015, opacity: 0.7 },
			{ x: 6, y: -2, scaleX: 0.99, opacity: 0.6 },
			{ x: -2, y: 3, scaleX: 1.008, opacity: 0.75 },
		];

		const tl = gsap.timeline();
		for (let i = 0; i < 5; i++) {
			const s = shifts[Math.floor(Math.random() * shifts.length)];
			tl.to([titleRedRef, titleBlueRef], {
				duration: 0.04,
				x: (idx: number) => (idx === 0 ? s.x : -s.x),
				y: (idx: number) => (idx === 0 ? s.y : -s.y),
				scaleX: s.scaleX,
				opacity: s.opacity,
				ease: 'none',
			}).to([titleRedRef, titleBlueRef], {
				duration: 0.04,
				x: 0, y: 0, scaleX: 1, opacity: 0.5,
				ease: 'none',
			});
		}

		if (glitchBarsRef) {
			const bars = glitchBarsRef.querySelectorAll<HTMLElement>('.glitch-bar');
			bars.forEach((bar) => {
				gsap.set(bar, {
					scaleX: Math.random() * 0.6 + 0.1,
					top: `${Math.random() * 100}%`,
					opacity: Math.random() * 0.5 + 0.1,
				});
				gsap.to(bar, { opacity: 0, duration: 0.15, ease: 'none' });
			});
		}
	}

	onMount(() => {
		const c = noiseCanvas.getContext('2d')!;
		noiseCanvas.width = 256;
		noiseCanvas.height = 256;

		function drawNoise() {
			const img = c.createImageData(256, 256);
			for (let i = 0; i < img.data.length; i += 4) {
				const v = Math.random() * 255;
				img.data[i] = v;
				img.data[i + 1] = v;
				img.data[i + 2] = v;
				img.data[i + 3] = Math.random() * 18;
			}
			c.putImageData(img, 0, 0);
			requestAnimationFrame(drawNoise);
		}
		drawNoise();

		gsap.set([titleRef, titleRedRef, titleBlueRef, subtitleRef, ctaRef, statusRef], { autoAlpha: 0 });

		const tl = gsap.timeline({ delay: 0.2 });

		tl.to(scanlineRef, { top: '100%', duration: 1.0, ease: 'power2.inOut' })
			.to(titleRef, { autoAlpha: 1, duration: 0 }, '-=0.1')
			.to(titleRedRef, { autoAlpha: 0.5, duration: 0 }, '<')
			.to(titleBlueRef, { autoAlpha: 0.5, duration: 0 }, '<')
			.add(() => {
				scrambleText(titleRef, 'SYSTEM\nFAILURE', 1.4);
			})
			.to(statusRef, { autoAlpha: 1, duration: 0.4 }, '+=0.3')
			.to(subtitleRef, { autoAlpha: 1, y: 0, duration: 0.7, ease: 'power3.out' }, '+=0.2')
			.to(ctaRef, { autoAlpha: 1, duration: 0.6, ease: 'power2.out' }, '-=0.3');

		const glitchInterval = setInterval(triggerGlitch, 3500 + Math.random() * 2000);

		gsap.to(scanlineRef, {
			top: '-4px',
			duration: 6,
			repeat: -1,
			ease: 'none',
			delay: 2.5,
		});

		return () => {
			clearInterval(glitchInterval);
			gsap.killTweensOf([titleRef, titleRedRef, titleBlueRef, subtitleRef, ctaRef, scanlineRef]);
		};
	});
</script>

<section class="h2" bind:this={section}>
	<canvas class="h2__noise" bind:this={noiseCanvas}></canvas>

	<div class="h2__scanline" bind:this={scanlineRef}></div>

	<div class="h2__glitch-bars" bind:this={glitchBarsRef} aria-hidden="true">
		{#each { length: 8 } as _}
			<div class="glitch-bar"></div>
		{/each}
	</div>

	<div class="h2__grid"></div>

	<div class="h2__content">
		<div class="h2__status" bind:this={statusRef}>
			<Warning size={14} weight="fill" />
			<span class="h2__status-dot"></span>
			<span>CRITICAL · NODE_BREACH_0x4F2A</span>
		</div>

		<div class="h2__title-wrap">
			<pre class="h2__title-ghost h2__title-red" bind:this={titleRedRef} aria-hidden="true">SYSTEM{'\n'}FAILURE</pre>
			<pre class="h2__title-ghost h2__title-blue" bind:this={titleBlueRef} aria-hidden="true">SYSTEM{'\n'}FAILURE</pre>
			<pre class="h2__title" bind:this={titleRef}>SYSTEM{'\n'}FAILURE</pre>
		</div>

		<p class="h2__subtitle" bind:this={subtitleRef}>
			<Eye size={14} weight="fill" />
			&nbsp;All systems compromised. Initiating emergency protocol.
		</p>

		<div class="h2__cta" bind:this={ctaRef}>
			<button class="h2__btn">
				<span class="h2__btn-bracket">[</span>
				INITIATE SEQUENCE
				<span class="h2__btn-bracket">]</span>
			</button>
			<span class="h2__countdown">T-00:04:32</span>
		</div>
	</div>

	<div class="h2__corners">
		<span class="corner tl"></span>
		<span class="corner tr"></span>
		<span class="corner bl"></span>
		<span class="corner br"></span>
	</div>

	<div class="h2__vignette"></div>
</section>

<style>
	.h2 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #050508;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.h2__noise {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		object-fit: cover;
		opacity: 0.045;
		pointer-events: none;
		z-index: 1;
		image-rendering: pixelated;
	}

	.h2__grid {
		position: absolute;
		inset: 0;
		background-image:
			linear-gradient(rgba(255, 0, 60, 0.04) 1px, transparent 1px),
			linear-gradient(90deg, rgba(255, 0, 60, 0.04) 1px, transparent 1px);
		background-size: 60px 60px;
		z-index: 1;
	}

	.h2__scanline {
		position: absolute;
		left: 0;
		right: 0;
		top: -4px;
		height: 3px;
		background: linear-gradient(90deg, transparent, rgba(255, 60, 60, 0.9), rgba(255,255,255,0.6), rgba(255, 60, 60, 0.9), transparent);
		box-shadow: 0 0 20px rgba(255, 60, 60, 0.8), 0 0 60px rgba(255, 60, 60, 0.3);
		z-index: 10;
		pointer-events: none;
	}

	.h2__glitch-bars {
		position: absolute;
		inset: 0;
		z-index: 9;
		pointer-events: none;
	}

	:global(.glitch-bar) {
		position: absolute;
		left: 0;
		right: 0;
		height: 6px;
		background: rgba(255, 0, 60, 0.5);
		opacity: 0;
		mix-blend-mode: screen;
	}

	.h2__content {
		position: relative;
		z-index: 5;
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 2rem;
	}

	.h2__status {
		display: inline-flex;
		align-items: center;
		gap: 0.5rem;
		font-family: var(--font-mono);
		font-size: 0.68rem;
		letter-spacing: 0.14em;
		color: rgba(255, 80, 80, 0.9);
		text-transform: uppercase;
		padding: 0.35rem 0.9rem;
		border: 1px solid rgba(255, 60, 60, 0.3);
		border-radius: 3px;
		background: rgba(255, 0, 0, 0.05);
	}

	.h2__status-dot {
		width: 6px;
		height: 6px;
		border-radius: 50%;
		background: #ff3c3c;
		animation: blink 0.8s step-end infinite;
	}

	@keyframes blink {
		0%, 100% { opacity: 1; }
		50% { opacity: 0; }
	}

	.h2__title-wrap {
		position: relative;
		line-height: 1;
	}

	.h2__title,
	.h2__title-ghost {
		font-family: var(--font-cinematic);
		font-size: clamp(5.5rem, 16vw, 16rem);
		line-height: 0.88;
		letter-spacing: 0.04em;
		white-space: pre;
		text-align: center;
	}

	.h2__title {
		color: #f5f5f5;
		position: relative;
		z-index: 2;
		text-shadow:
			0 0 30px rgba(255, 80, 80, 0.4),
			0 0 80px rgba(255, 40, 40, 0.15);
	}

	.h2__title-ghost {
		position: absolute;
		inset: 0;
		z-index: 1;
		mix-blend-mode: screen;
	}

	.h2__title-red {
		color: #ff2244;
		transform: translateX(-4px);
	}

	.h2__title-blue {
		color: #22aaff;
		transform: translateX(4px);
	}

	.h2__subtitle {
		display: flex;
		align-items: center;
		justify-content: center;
		font-family: var(--font-mono);
		font-size: 0.8rem;
		color: rgba(200, 180, 180, 0.7);
		letter-spacing: 0.06em;
		gap: 0.3rem;
	}

	.h2__cta {
		display: flex;
		align-items: center;
		gap: 1.5rem;
	}

	.h2__btn {
		font-family: var(--font-mono);
		font-size: 0.82rem;
		letter-spacing: 0.14em;
		color: #ff4444;
		background: transparent;
		border: 1px solid rgba(255, 60, 60, 0.5);
		padding: 0.7rem 1.6rem;
		cursor: pointer;
		text-transform: uppercase;
		transition: background 0.2s, box-shadow 0.2s;
		position: relative;
		overflow: hidden;
	}

	.h2__btn::before {
		content: '';
		position: absolute;
		inset: 0;
		background: rgba(255, 60, 60, 0.08);
		transform: scaleX(0);
		transform-origin: left;
		transition: transform 0.25s ease;
	}

	.h2__btn:hover::before { transform: scaleX(1); }
	.h2__btn:hover {
		box-shadow: 0 0 30px rgba(255, 60, 60, 0.35);
	}

	.h2__btn-bracket {
		opacity: 0.5;
		margin: 0 0.2rem;
	}

	.h2__countdown {
		font-family: var(--font-mono);
		font-size: 1.1rem;
		color: rgba(255, 80, 80, 0.6);
		letter-spacing: 0.1em;
		animation: countdownPulse 1s ease-in-out infinite;
	}

	@keyframes countdownPulse {
		0%, 100% { opacity: 0.6; }
		50% { opacity: 1; color: rgba(255, 120, 120, 0.9); }
	}

	.h2__corners {
		position: absolute;
		inset: 24px;
		pointer-events: none;
		z-index: 4;
	}

	.corner {
		position: absolute;
		width: 24px;
		height: 24px;
		border-color: rgba(255, 60, 60, 0.5);
		border-style: solid;
	}

	.corner.tl { top: 0; left: 0; border-width: 2px 0 0 2px; }
	.corner.tr { top: 0; right: 0; border-width: 2px 2px 0 0; }
	.corner.bl { bottom: 0; left: 0; border-width: 0 0 2px 2px; }
	.corner.br { bottom: 0; right: 0; border-width: 0 2px 2px 0; }

	.h2__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at center, transparent 45%, rgba(0,0,0,0.92) 100%);
		pointer-events: none;
		z-index: 3;
	}
</style>
