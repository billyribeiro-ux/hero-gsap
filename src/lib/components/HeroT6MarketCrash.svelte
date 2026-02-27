<script lang="ts">
	import { onMount } from 'svelte';
	import { gsap } from 'gsap';
	import { ArrowFatDown, ArrowFatUp, Warning } from 'phosphor-svelte';

	let section: HTMLElement;
	let canvasLeft: HTMLCanvasElement;
	let canvasRight: HTMLCanvasElement;
	let canvasTicker: HTMLCanvasElement;
	let titleRef: HTMLElement;
	let leftLabel: HTMLElement;
	let rightLabel: HTMLElement;
	let dividerRef: HTMLElement;
	let ctaRef: HTMLElement;

	onMount(() => {
		const ctxL = canvasLeft.getContext('2d')!;
		const ctxR = canvasRight.getContext('2d')!;
		const ctxT = canvasTicker.getContext('2d')!;
		let W = window.innerWidth;
		let H = window.innerHeight;
		let raf: number;
		let time = 0;

		function setSize() {
			W = window.innerWidth;
			H = window.innerHeight;
			canvasLeft.width = W / 2;
			canvasLeft.height = H;
			canvasRight.width = W / 2;
			canvasRight.height = H;
			canvasTicker.width = W;
			canvasTicker.height = 36;
		}
		setSize();

		// Generate crash curve (down+volatile)
		function genCrash(start: number, count: number): number[] {
			const pts: number[] = [start];
			let v = start;
			for (let i = 1; i < count; i++) {
				v += -(Math.random() * 3.2 + 0.5) + Math.random() * 1.4;
				v += Math.sin(i * 0.3) * 2;
				pts.push(v);
			}
			return pts;
		}

		// Generate pump curve (up+volatile)
		function genPump(start: number, count: number): number[] {
			const pts: number[] = [start];
			let v = start;
			for (let i = 1; i < count; i++) {
				v += (Math.random() * 3.2 + 0.5) - Math.random() * 1.2;
				v += Math.sin(i * 0.25) * 1.5;
				pts.push(v);
			}
			return pts;
		}

		const CRASH_PTS = 120;
		let crashCurve = genCrash(100, CRASH_PTS);
		let pumpCurve = genPump(100, CRASH_PTS);
		let crashOffset = 0;
		let pumpOffset = 0;

		// Explosion particles for crash side
		type Debris = { x: number; y: number; vx: number; vy: number; alpha: number; size: number };
		const debris: Debris[] = [];
		function spawnDebris(x: number, y: number) {
			for (let i = 0; i < 12; i++) {
				const angle = Math.random() * Math.PI * 2;
				const speed = 1 + Math.random() * 5;
				debris.push({
					x, y,
					vx: Math.cos(angle) * speed,
					vy: Math.sin(angle) * speed,
					alpha: 1,
					size: 2 + Math.random() * 5,
				});
			}
		}

		// Pump fireworks
		type Firework = { x: number; y: number; vx: number; vy: number; alpha: number; size: number; hue: number };
		const fireworks: Firework[] = [];
		function spawnFirework(x: number, y: number) {
			for (let i = 0; i < 18; i++) {
				const angle = (i / 18) * Math.PI * 2;
				const speed = 1.5 + Math.random() * 4;
				fireworks.push({
					x, y,
					vx: Math.cos(angle) * speed,
					vy: Math.sin(angle) * speed,
					alpha: 1,
					size: 2 + Math.random() * 3,
					hue: 80 + Math.random() * 60,
				});
			}
		}

		// Spawn periodically
		const debrisInterval = setInterval(() => {
			const halfW = W / 2;
			const y = H * 0.3 + Math.random() * H * 0.4;
			spawnDebris(Math.random() * halfW, y);
		}, 900);

		const fwInterval = setInterval(() => {
			const halfW = W / 2;
			spawnFirework(halfW * 0.2 + Math.random() * halfW * 0.6, H * 0.15 + Math.random() * H * 0.55);
		}, 700);

		// Ticker data
		const crashTickers = ['GME -44%','TSLA -12%','LUNA -99%','MSTR -28%','RIOT -33%','COIN -18%','SNAP -37%','META -24%','NVDA -11%','AMD -16%'];
		const pumpTickers  = ['NVDA +44%','SMCI +28%','ARM +18%','AI +33%','PLTR +22%','MRVL +19%','ANET +15%','CRWD +24%','MU +17%','ORCL +12%'];
		let tickerOffset = 0;

		function drawSideBg(c: CanvasRenderingContext2D, w: number, h: number, crash: boolean) {
			c.clearRect(0, 0, w, h);
			const bg = c.createLinearGradient(0, 0, 0, h);
			if (crash) {
				bg.addColorStop(0, '#180404');
				bg.addColorStop(0.5, '#0d0202');
				bg.addColorStop(1, '#08010a');
			} else {
				bg.addColorStop(0, '#041808');
				bg.addColorStop(0.5, '#020d04');
				bg.addColorStop(1, '#010802');
			}
			c.fillStyle = bg;
			c.fillRect(0, 0, w, h);
		}

		function drawPriceCurve(
			c: CanvasRenderingContext2D,
			pts: number[], w: number, h: number,
			crash: boolean, offset: number
		) {
			if (pts.length < 2) return;
			const visible = pts.slice(Math.floor(offset), Math.floor(offset) + 80);
			if (visible.length < 2) return;

			const minV = Math.min(...visible);
			const maxV = Math.max(...visible);
			const range = maxV - minV || 1;
			const padT = h * 0.2, padB = h * 0.15;
			const chartH = h - padT - padB;

			const toY = (v: number) => padT + chartH - ((v - minV) / range) * chartH;

			// Grid
			for (let g = 0; g <= 4; g++) {
				const gy = padT + (g / 4) * chartH;
				c.beginPath();
				c.moveTo(0, gy);
				c.lineTo(w, gy);
				c.strokeStyle = crash ? 'rgba(255,40,40,0.08)' : 'rgba(0,200,80,0.08)';
				c.lineWidth = 1;
				c.stroke();
			}

			// Area fill
			c.beginPath();
			visible.forEach((v, i) => {
				const x = (i / (visible.length - 1)) * w;
				const y = toY(v);
				i === 0 ? c.moveTo(x, y) : c.lineTo(x, y);
			});
			c.lineTo(w, h);
			c.lineTo(0, h);
			c.closePath();
			const areaGrad = c.createLinearGradient(0, padT, 0, h);
			if (crash) {
				areaGrad.addColorStop(0, 'rgba(255,40,40,0.3)');
				areaGrad.addColorStop(1, 'rgba(255,40,40,0.02)');
			} else {
				areaGrad.addColorStop(0, 'rgba(0,200,80,0.28)');
				areaGrad.addColorStop(1, 'rgba(0,200,80,0.02)');
			}
			c.fillStyle = areaGrad;
			c.fill();

			// Line
			c.beginPath();
			visible.forEach((v, i) => {
				const x = (i / (visible.length - 1)) * w;
				const y = toY(v);
				i === 0 ? c.moveTo(x, y) : c.lineTo(x, y);
			});
			c.strokeStyle = crash ? 'rgba(255,60,60,0.9)' : 'rgba(0,210,80,0.9)';
			c.lineWidth = 2.5;
			c.shadowColor = crash ? 'rgba(255,60,60,0.6)' : 'rgba(0,210,80,0.6)';
			c.shadowBlur = 8;
			c.stroke();
			c.shadowBlur = 0;

			// Live point glow
			const lx = w;
			const lv = visible[visible.length - 1];
			const ly = toY(lv);
			const pulse = 0.6 + Math.sin(time * 7) * 0.4;
			c.beginPath();
			c.arc(lx - 2, ly, 5, 0, Math.PI * 2);
			c.fillStyle = crash ? '#ff3c50' : '#00d264';
			c.shadowColor = crash ? '#ff3c50' : '#00d264';
			c.shadowBlur = 14 * pulse;
			c.fill();
			c.shadowBlur = 0;

			// Price label
			const lastPrice = (visible[visible.length - 1] * (crash ? 420 : 420)).toFixed(2);
			c.fillStyle = crash ? 'rgba(255,80,80,0.9)' : 'rgba(0,210,80,0.9)';
			c.font = 'bold 11px var(--font-mono)';
			c.textAlign = 'right';
			c.fillText('$' + lastPrice, w - 8, ly - 8);
		}

		function drawDebris(c: CanvasRenderingContext2D) {
			for (let i = debris.length - 1; i >= 0; i--) {
				const d = debris[i];
				d.x += d.vx;
				d.y += d.vy;
				d.vy += 0.15;
				d.alpha -= 0.012;
				if (d.alpha <= 0) { debris.splice(i, 1); continue; }
				c.globalAlpha = d.alpha;
				c.fillStyle = `rgba(255,${40 + Math.random() * 60},0,1)`;
				c.beginPath();
				c.arc(d.x, d.y, d.size, 0, Math.PI * 2);
				c.fill();
			}
			c.globalAlpha = 1;
		}

		function drawFireworks(c: CanvasRenderingContext2D, offX: number) {
			for (let i = fireworks.length - 1; i >= 0; i--) {
				const fw = fireworks[i];
				fw.x += fw.vx;
				fw.y += fw.vy;
				fw.vy += 0.05;
				fw.alpha -= 0.013;
				if (fw.alpha <= 0) { fireworks.splice(i, 1); continue; }
				c.globalAlpha = fw.alpha;
				c.fillStyle = `hsla(${fw.hue},90%,65%,1)`;
				c.shadowColor = `hsla(${fw.hue},90%,65%,0.8)`;
				c.shadowBlur = 6;
				c.beginPath();
				c.arc(fw.x - offX, fw.y, fw.size, 0, Math.PI * 2);
				c.fill();
			}
			c.globalAlpha = 1;
			c.shadowBlur = 0;
		}

		function drawRedFlicker(c: CanvasRenderingContext2D, w: number, h: number) {
			// Random edge flicker on crash side
			if (Math.random() > 0.85) {
				const flicker = Math.random() * 0.08;
				const fg = c.createRadialGradient(w * 0.5, h * 0.4, 0, w * 0.5, h * 0.4, w * 0.6);
				fg.addColorStop(0, 'rgba(255,30,30,0)');
				fg.addColorStop(0.7, `rgba(255,30,30,${flicker})`);
				fg.addColorStop(1, `rgba(255,30,30,${flicker * 2})`);
				c.fillStyle = fg;
				c.fillRect(0, 0, w, h);
			}
		}

		function drawTicker2() {
			const halfW = W / 2;
			ctxT.clearRect(0, 0, W, 36);

			// Left (crash) ticker
			ctxT.fillStyle = 'rgba(20,4,4,0.97)';
			ctxT.fillRect(0, 0, halfW, 36);
			ctxT.strokeStyle = 'rgba(255,40,40,0.2)';
			ctxT.lineWidth = 1;
			ctxT.beginPath();
			ctxT.moveTo(0, 0); ctxT.lineTo(halfW, 0);
			ctxT.stroke();

			// Right (pump) ticker
			ctxT.fillStyle = 'rgba(4,20,6,0.97)';
			ctxT.fillRect(halfW, 0, halfW, 36);
			ctxT.strokeStyle = 'rgba(0,200,80,0.2)';
			ctxT.beginPath();
			ctxT.moveTo(halfW, 0); ctxT.lineTo(W, 0);
			ctxT.stroke();

			tickerOffset -= 1.2;
			const itemW = 100;
			const totalW = crashTickers.length * itemW;
			if (tickerOffset < -totalW) tickerOffset = 0;

			// Crash tickers (left half)
			ctxT.save();
			ctxT.beginPath();
			ctxT.rect(0, 0, halfW, 36);
			ctxT.clip();
			for (let rep = 0; rep < 4; rep++) {
				crashTickers.forEach((item, i) => {
					const tx = tickerOffset + rep * totalW + i * itemW;
					ctxT.fillStyle = 'rgba(255,70,70,0.8)';
					ctxT.font = '9px var(--font-mono)';
					ctxT.textAlign = 'left';
					ctxT.fillText(item, tx, 22);
				});
			}
			ctxT.restore();

			// Pump tickers (right half)
			ctxT.save();
			ctxT.beginPath();
			ctxT.rect(halfW, 0, halfW, 36);
			ctxT.clip();
			const pumpOffset2 = -tickerOffset;
			for (let rep = 0; rep < 4; rep++) {
				pumpTickers.forEach((item, i) => {
					const tx = halfW + ((pumpOffset2 + rep * totalW + i * itemW) % totalW);
					ctxT.fillStyle = 'rgba(0,220,90,0.8)';
					ctxT.font = '9px var(--font-mono)';
					ctxT.textAlign = 'left';
					ctxT.fillText(item, tx, 22);
				});
			}
			ctxT.restore();
		}

		function draw() {
			time += 0.016;
			const halfW = W / 2;

			// Scroll curves
			crashOffset += 0.18;
			pumpOffset += 0.14;
			if (crashOffset > crashCurve.length - 80) crashCurve = [...crashCurve, ...genCrash(crashCurve[crashCurve.length - 1], 30)];
			if (pumpOffset > pumpCurve.length - 80) pumpCurve = [...pumpCurve, ...genPump(pumpCurve[pumpCurve.length - 1], 30)];

			// Crash side
			drawSideBg(ctxL, halfW, H, true);
			drawRedFlicker(ctxL, halfW, H);
			drawPriceCurve(ctxL, crashCurve, halfW, H, true, crashOffset);
			drawDebris(ctxL);

			// Pump side
			drawSideBg(ctxR, halfW, H, false);
			drawPriceCurve(ctxR, pumpCurve, halfW, H, false, pumpOffset);
			drawFireworks(ctxR, halfW);

			drawTicker2();

			raf = requestAnimationFrame(draw);
		}
		draw();

		// Entrance
		gsap.set([titleRef, leftLabel, rightLabel, ctaRef], { autoAlpha: 0 });
		gsap.set(dividerRef, { scaleY: 0, transformOrigin: 'top' });
		gsap.set(leftLabel, { x: -60 });
		gsap.set(rightLabel, { x: 60 });

		const tl = gsap.timeline({ delay: 0.2 });
		tl.to(dividerRef, { scaleY: 1, duration: 0.8, ease: 'power3.out' })
			.to(leftLabel, { autoAlpha: 1, x: 0, duration: 0.7, ease: 'expo.out' }, '-=0.4')
			.to(rightLabel, { autoAlpha: 1, x: 0, duration: 0.7, ease: 'expo.out' }, '-=0.7')
			.to(titleRef, { autoAlpha: 1, duration: 0.6, ease: 'power2.out' }, '-=0.3')
			.to(ctaRef, { autoAlpha: 1, duration: 0.5, ease: 'power2.out' }, '-=0.2');

		// Divider pulse
		gsap.to(dividerRef, {
			boxShadow: '0 0 30px rgba(200,200,200,0.4)',
			duration: 1.5,
			repeat: -1,
			yoyo: true,
			ease: 'sine.inOut',
			delay: 1.5,
		});

		const onResize = () => {
			setSize();
		};
		window.addEventListener('resize', onResize);

		return () => {
			cancelAnimationFrame(raf);
			clearInterval(debrisInterval);
			clearInterval(fwInterval);
			window.removeEventListener('resize', onResize);
		};
	});
</script>

<section class="ht6" bind:this={section}>
	<canvas class="ht6__left-canvas" bind:this={canvasLeft}></canvas>
	<canvas class="ht6__right-canvas" bind:this={canvasRight}></canvas>
	<canvas class="ht6__ticker" bind:this={canvasTicker}></canvas>

	<div class="ht6__divider" bind:this={dividerRef}></div>

	<div class="ht6__label ht6__label-left" bind:this={leftLabel}>
		<ArrowFatDown size={28} weight="fill" />
		<div class="ht6__label-text">
			<span class="ht6__bear-title">MARKET</span>
			<span class="ht6__bear-title">CRASH</span>
			<span class="ht6__label-pct">-34.8%</span>
		</div>
		<Warning size={18} weight="fill" style="opacity:0.5" />
	</div>

	<div class="ht6__label ht6__label-right" bind:this={rightLabel}>
		<div class="ht6__label-text ht6__label-text-r">
			<span class="ht6__bull-title">BULL</span>
			<span class="ht6__bull-title">RUN</span>
			<span class="ht6__label-pct ht6__bull-pct">+284%</span>
		</div>
		<ArrowFatUp size={28} weight="fill" />
	</div>

	<div class="ht6__center-ui">
		<div class="ht6__title" bind:this={titleRef}>
			<span>WHICH SIDE</span>
			<span class="ht6__title-q">ARE YOU ON?</span>
		</div>
		<div class="ht6__cta" bind:this={ctaRef}>
			<button class="ht6__btn-bear">
				<ArrowFatDown size={13} weight="fill" /> SHORT
			</button>
			<button class="ht6__btn-bull">
				LONG <ArrowFatUp size={13} weight="fill" />
			</button>
		</div>
	</div>

	<div class="ht6__scanlines"></div>
</section>

<style>
	.ht6 {
		position: relative;
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #0a0404;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.ht6__left-canvas {
		position: absolute;
		top: 0; left: 0;
		width: 50%; height: 100%;
		z-index: 2;
	}

	.ht6__right-canvas {
		position: absolute;
		top: 0; right: 0;
		width: 50%; height: 100%;
		z-index: 2;
	}

	.ht6__ticker {
		position: absolute;
		bottom: 0; left: 0;
		width: 100%;
		height: 36px;
		z-index: 12;
	}

	.ht6__divider {
		position: absolute;
		top: 0; bottom: 36px;
		left: 50%;
		width: 2px;
		background: linear-gradient(
			180deg,
			rgba(200,200,200,0) 0%,
			rgba(220,220,220,0.7) 30%,
			rgba(255,255,255,0.9) 50%,
			rgba(220,220,220,0.7) 70%,
			rgba(200,200,200,0) 100%
		);
		transform: translateX(-50%);
		z-index: 15;
	}

	.ht6__label {
		position: absolute;
		top: 50%;
		transform: translateY(-50%);
		z-index: 10;
		display: flex;
		align-items: center;
		gap: 1rem;
	}

	.ht6__label-left {
		left: 3%;
		color: rgba(255, 70, 70, 0.85);
	}

	.ht6__label-right {
		right: 3%;
		color: rgba(0, 210, 100, 0.85);
	}

	.ht6__label-text {
		display: flex;
		flex-direction: column;
		line-height: 0.9;
	}

	.ht6__label-text-r {
		align-items: flex-end;
	}

	.ht6__bear-title, .ht6__bull-title {
		font-family: var(--font-cinematic);
		font-size: clamp(2rem, 4.5vw, 5rem);
		letter-spacing: 0.08em;
	}

	.ht6__bear-title { color: rgba(255,60,60,0.9); text-shadow: 0 0 30px rgba(255,40,40,0.5); }
	.ht6__bull-title { color: rgba(0,210,80,0.9); text-shadow: 0 0 30px rgba(0,200,80,0.5); }

	.ht6__label-pct {
		font-family: var(--font-mono);
		font-size: clamp(1rem, 2.2vw, 1.8rem);
		font-weight: 700;
		color: rgba(255,80,80,0.85);
		letter-spacing: 0.06em;
		margin-top: 0.2rem;
	}

	.ht6__bull-pct { color: rgba(0,220,90,0.85) !important; }

	.ht6__center-ui {
		position: relative;
		z-index: 20;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1.4rem;
		text-align: center;
	}

	.ht6__title {
		display: flex;
		flex-direction: column;
		align-items: center;
		font-family: var(--font-cinematic);
		font-size: clamp(1rem, 2.5vw, 2.2rem);
		letter-spacing: 0.18em;
		color: rgba(220,220,220,0.8);
		line-height: 1.1;
	}

	.ht6__title-q {
		color: rgba(255,255,255,0.95);
		text-shadow: 0 0 20px rgba(255,255,255,0.3);
	}

	.ht6__cta {
		display: flex;
		gap: 0.8rem;
	}

	.ht6__btn-bear, .ht6__btn-bull {
		display: inline-flex;
		align-items: center;
		gap: 0.4rem;
		padding: 0.6rem 1.2rem;
		font-family: var(--font-mono);
		font-size: 0.75rem;
		font-weight: 700;
		letter-spacing: 0.1em;
		cursor: pointer;
		transition: all 0.2s;
		border: none;
	}

	.ht6__btn-bear {
		background: rgba(255,50,50,0.2);
		border: 1px solid rgba(255,60,60,0.5);
		color: rgba(255,100,100,0.95);
		box-shadow: 0 0 20px rgba(255,40,40,0.15);
	}
	.ht6__btn-bear:hover {
		background: rgba(255,50,50,0.38);
		box-shadow: 0 0 40px rgba(255,40,40,0.4);
		transform: translateY(-2px);
	}

	.ht6__btn-bull {
		background: rgba(0,200,80,0.18);
		border: 1px solid rgba(0,210,80,0.5);
		color: rgba(80,230,120,0.95);
		box-shadow: 0 0 20px rgba(0,200,80,0.12);
	}
	.ht6__btn-bull:hover {
		background: rgba(0,200,80,0.35);
		box-shadow: 0 0 40px rgba(0,200,80,0.4);
		transform: translateY(-2px);
	}

	.ht6__scanlines {
		position: absolute;
		inset: 0;
		background: repeating-linear-gradient(
			0deg,
			transparent, transparent 3px,
			rgba(0,0,0,0.04) 3px, rgba(0,0,0,0.04) 4px
		);
		pointer-events: none;
		z-index: 18;
	}
</style>
