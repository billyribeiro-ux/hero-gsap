<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { Terminal, CaretRight, CheckCircle } from 'phosphor-svelte';

	let section: HTMLElement;
	let titleRef: HTMLElement;
	let terminalRef: HTMLElement;
	let ctaRef: HTMLElement;
	let cursorRef: HTMLElement;
	let progressBar: HTMLElement;
	let progressFill: HTMLElement;

	const CHARS = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789@#$%&';

	const lines = [
		{ prefix: '>', text: 'INITIALIZING NEURAL CORE...', delay: 0.3, color: '#4ade80' },
		{ prefix: '>', text: 'LOADING COGNITION MATRIX [████████] 100%', delay: 1.1, color: '#60a5fa' },
		{ prefix: '>', text: 'BINDING SYNAPTIC PATHWAYS...', delay: 2.0, color: '#a78bfa' },
		{ prefix: '>', text: 'CONSCIOUSNESS ONLINE.', delay: 2.9, color: '#fff' },
	];

	function typeText(el: HTMLElement, text: string, speed = 38): Promise<void> {
		return new Promise((resolve) => {
			let i = 0;
			const iv = setInterval(() => {
				el.textContent = text.slice(0, ++i);
				if (i >= text.length) { clearInterval(iv); resolve(); }
			}, speed);
		});
	}

	function scrambleReveal(el: HTMLElement, finalText: string, duration = 1.6): Promise<void> {
		return new Promise((resolve) => {
			const total = finalText.length * 5;
			let iter = 0;
			const iv = setInterval(() => {
				el.textContent = finalText
					.split('')
					.map((c, i) => {
						if (c === ' ' || c === '\n') return c;
						if (i < Math.floor(iter / 5)) return c;
						return CHARS[Math.floor(Math.random() * CHARS.length)];
					})
					.join('');
				iter++;
				if (iter >= total) { el.textContent = finalText; clearInterval(iv); resolve(); }
			}, (duration * 1000) / total);
		});
	}

	onMount(() => {
		gsap.set([titleRef, terminalRef, ctaRef], { autoAlpha: 0 });
		gsap.set(progressFill, { scaleX: 0, transformOrigin: 'left center' });

		const tl = gsap.timeline({ delay: 0.3 });

		tl.to(progressFill, {
			scaleX: 1,
			duration: 1.8,
			ease: 'power2.inOut',
		})
		.to(progressBar, { autoAlpha: 0, duration: 0.3 }, '+=0.2')
		.to(terminalRef, { autoAlpha: 1, duration: 0.4, ease: 'power2.out' }, '-=0.1')
		.add(() => {
			void (async () => {
				const lineEls = terminalRef.querySelectorAll<HTMLElement>('.h5__line-text');
				for (let i = 0; i < lines.length; i++) {
					await new Promise<void>((res) => setTimeout(res, lines[i].delay * 300));
					const el = lineEls[i];
					if (el) {
						el.style.color = lines[i].color;
						await typeText(el, lines[i].text, 22);
					}
				}
				gsap.to(titleRef, { autoAlpha: 1, duration: 0.8, ease: 'power2.out', delay: 0.3 });
				gsap.to(ctaRef, { autoAlpha: 1, duration: 0.7, ease: 'power2.out', delay: 0.7 });
				if (titleRef) {
					await new Promise<void>((res) => setTimeout(res, 400));
					await scrambleReveal(titleRef.querySelector('.h5__big') as HTMLElement, 'AWAKENING', 2);
				}
			})();
		});

		gsap.to(cursorRef, {
			autoAlpha: 0,
			duration: 0.5,
			repeat: -1,
			yoyo: true,
			ease: 'none',
		});

		const mousemove = (e: MouseEvent) => {
			const rect = section.getBoundingClientRect();
			const mx = (e.clientX - rect.left) / rect.width - 0.5;
			const my = (e.clientY - rect.top) / rect.height - 0.5;
			gsap.to('.h5__grid', {
				backgroundPositionX: `${50 + mx * 10}%`,
				backgroundPositionY: `${50 + my * 10}%`,
				duration: 2,
				ease: 'power2.out',
			});
		};
		section.addEventListener('mousemove', mousemove);

		return () => {
			section.removeEventListener('mousemove', mousemove);
		};
	});
</script>

<section class="h5" bind:this={section}>
	<div class="h5__grid"></div>
	<div class="h5__scanlines"></div>

	<div class="h5__glow-orb h5__glow-orb-1"></div>
	<div class="h5__glow-orb h5__glow-orb-2"></div>

	<div class="h5__progress-wrap" bind:this={progressBar}>
		<div class="h5__progress-label">
			<Terminal size={13} weight="fill" />
			<span>BOOTING SYSTEM</span>
		</div>
		<div class="h5__progress-track">
			<div class="h5__progress-fill" bind:this={progressFill}></div>
		</div>
	</div>

	<div class="h5__layout">
		<div class="h5__terminal-panel" bind:this={terminalRef}>
			<div class="h5__term-header">
				<div class="h5__term-dots">
					<span class="dot red"></span>
					<span class="dot yellow"></span>
					<span class="dot green"></span>
				</div>
				<span class="h5__term-title">neural_init.sh</span>
				<Terminal size={13} weight="fill" />
			</div>
			<div class="h5__term-body">
				{#each lines as line, i}
					<div class="h5__term-line">
						<CaretRight size={11} weight="fill" class="h5__prompt-icon" />
						<span class="h5__line-text"></span>
					</div>
				{/each}
				<div class="h5__term-line h5__term-cursor-line">
					<CaretRight size={11} weight="fill" class="h5__prompt-icon" />
					<span bind:this={cursorRef} class="h5__cursor">█</span>
				</div>
			</div>
		</div>

		<div class="h5__hero-text" bind:this={titleRef}>
			<div class="h5__badge">
				<CheckCircle size={13} weight="fill" />
				<span>SYSTEM ONLINE</span>
			</div>
			<div class="h5__big-wrap">
				<span class="h5__big">AWAKENING</span>
			</div>
			<p class="h5__desc">
				Artificial consciousness — one prompt at a time.
				<br/>The machine that writes back.
			</p>
		</div>
	</div>

	<div class="h5__cta" bind:this={ctaRef}>
		<button class="h5__btn">
			<Terminal size={16} weight="fill" />
			<span>Run Sequence</span>
		</button>
		<span class="h5__version">v9.4.1-alpha</span>
	</div>

	<div class="h5__vignette"></div>
</section>

<style>
	.h5 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #030a06;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 3rem;
	}

	.h5__grid {
		position: absolute;
		inset: 0;
		background-image:
			linear-gradient(rgba(74, 222, 128, 0.06) 1px, transparent 1px),
			linear-gradient(90deg, rgba(74, 222, 128, 0.06) 1px, transparent 1px);
		background-size: 44px 44px;
		z-index: 1;
		transition: background-position 2s;
	}

	.h5__scanlines {
		position: absolute;
		inset: 0;
		background: repeating-linear-gradient(
			0deg,
			transparent,
			transparent 3px,
			rgba(0, 0, 0, 0.07) 3px,
			rgba(0, 0, 0, 0.07) 4px
		);
		z-index: 2;
		pointer-events: none;
	}

	.h5__glow-orb {
		position: absolute;
		border-radius: 50%;
		filter: blur(80px);
		pointer-events: none;
		z-index: 1;
	}

	.h5__glow-orb-1 {
		width: 400px; height: 400px;
		background: rgba(20, 160, 80, 0.12);
		top: -100px; left: -100px;
		animation: orbFloat1 8s ease-in-out infinite;
	}

	.h5__glow-orb-2 {
		width: 350px; height: 350px;
		background: rgba(80, 220, 140, 0.08);
		bottom: -80px; right: -80px;
		animation: orbFloat2 10s ease-in-out infinite;
	}

	@keyframes orbFloat1 {
		0%, 100% { transform: translate(0, 0); }
		50% { transform: translate(60px, 40px); }
	}

	@keyframes orbFloat2 {
		0%, 100% { transform: translate(0, 0); }
		50% { transform: translate(-50px, -30px); }
	}

	.h5__progress-wrap {
		position: absolute;
		inset: 0;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 1.2rem;
		z-index: 10;
		background: #030a06;
	}

	.h5__progress-label {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		font-family: var(--font-mono);
		font-size: 0.72rem;
		letter-spacing: 0.15em;
		color: rgba(74, 222, 128, 0.8);
		text-transform: uppercase;
	}

	.h5__progress-track {
		width: 320px;
		height: 3px;
		background: rgba(74, 222, 128, 0.12);
		border-radius: 99px;
		overflow: hidden;
	}

	.h5__progress-fill {
		height: 100%;
		background: linear-gradient(90deg, #4ade80, #86efac);
		border-radius: 99px;
		box-shadow: 0 0 12px rgba(74, 222, 128, 0.6);
	}

	.h5__layout {
		position: relative;
		z-index: 5;
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 4rem;
		align-items: center;
		max-width: 1100px;
		width: 100%;
		padding: 0 4rem;
	}

	.h5__terminal-panel {
		background: rgba(5, 14, 8, 0.95);
		border: 1px solid rgba(74, 222, 128, 0.2);
		border-radius: 10px;
		overflow: hidden;
		box-shadow: 0 0 40px rgba(74, 222, 128, 0.1), 0 20px 60px rgba(0,0,0,0.5);
		backdrop-filter: blur(12px);
	}

	.h5__term-header {
		display: flex;
		align-items: center;
		gap: 0.6rem;
		padding: 0.7rem 1rem;
		border-bottom: 1px solid rgba(74, 222, 128, 0.12);
		background: rgba(74, 222, 128, 0.04);
	}

	.h5__term-dots {
		display: flex;
		gap: 0.35rem;
	}

	.dot {
		width: 10px; height: 10px;
		border-radius: 50%;
	}

	.dot.red { background: #ff5f57; }
	.dot.yellow { background: #ffbd2e; }
	.dot.green { background: #28ca41; }

	.h5__term-title {
		font-family: var(--font-mono);
		font-size: 0.72rem;
		color: rgba(74, 222, 128, 0.5);
		letter-spacing: 0.05em;
		flex: 1;
		text-align: center;
	}

	.h5__term-body {
		padding: 1.2rem 1.4rem;
		display: flex;
		flex-direction: column;
		gap: 0.6rem;
		min-height: 180px;
	}

	.h5__term-line {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		font-family: var(--font-mono);
		font-size: 0.78rem;
		color: rgba(200, 240, 210, 0.8);
	}

	:global(.h5__prompt-icon) {
		color: rgba(74, 222, 128, 0.6);
		flex-shrink: 0;
	}

	.h5__cursor {
		color: #4ade80;
		font-family: var(--font-mono);
		font-size: 0.85rem;
	}

	.h5__hero-text {
		display: flex;
		flex-direction: column;
		gap: 1.2rem;
	}

	.h5__badge {
		display: inline-flex;
		align-items: center;
		gap: 0.45rem;
		font-family: var(--font-mono);
		font-size: 0.68rem;
		letter-spacing: 0.14em;
		color: #4ade80;
		text-transform: uppercase;
	}

	.h5__big-wrap {
		line-height: 1;
	}

	.h5__big {
		font-family: var(--font-cinematic);
		font-size: clamp(4rem, 8vw, 8rem);
		letter-spacing: 0.06em;
		color: #fff;
		text-shadow:
			0 0 30px rgba(74, 222, 128, 0.5),
			0 0 80px rgba(74, 222, 128, 0.2);
	}

	.h5__desc {
		font-size: 0.92rem;
		color: rgba(160, 200, 170, 0.65);
		line-height: 1.75;
		font-weight: 300;
	}

	.h5__cta {
		position: relative;
		z-index: 5;
		display: flex;
		align-items: center;
		gap: 1.5rem;
	}

	.h5__btn {
		display: inline-flex;
		align-items: center;
		gap: 0.55rem;
		padding: 0.8rem 1.9rem;
		background: rgba(74, 222, 128, 0.12);
		border: 1px solid rgba(74, 222, 128, 0.4);
		border-radius: 6px;
		color: #4ade80;
		font-family: var(--font-mono);
		font-size: 0.82rem;
		letter-spacing: 0.08em;
		cursor: pointer;
		transition: all 0.25s;
		box-shadow: 0 0 20px rgba(74, 222, 128, 0.1);
	}

	.h5__btn:hover {
		background: rgba(74, 222, 128, 0.22);
		box-shadow: 0 0 40px rgba(74, 222, 128, 0.3);
		transform: translateY(-1px);
	}

	.h5__version {
		font-family: var(--font-mono);
		font-size: 0.68rem;
		color: rgba(74, 222, 128, 0.35);
		letter-spacing: 0.1em;
	}

	.h5__vignette {
		position: absolute;
		inset: 0;
		background: radial-gradient(ellipse at center, transparent 40%, rgba(3, 10, 6, 0.88) 100%);
		pointer-events: none;
		z-index: 4;
	}

	@media (max-width: 860px) {
		.h5__layout {
			grid-template-columns: 1fr;
			padding: 0 2rem;
			gap: 2rem;
		}
	}
</style>
