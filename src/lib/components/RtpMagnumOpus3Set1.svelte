<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { gsap } from 'gsap';
	import { ScrollTrigger } from 'gsap/ScrollTrigger';

	gsap.registerPlugin(ScrollTrigger);

	let mx = 0, my = 0;
	let s1a: HTMLCanvasElement, s1b: HTMLCanvasElement, s1c: HTMLCanvasElement, s1d: HTMLCanvasElement;
	let s2a: HTMLCanvasElement, s2b: HTMLCanvasElement;
	let s3a: HTMLCanvasElement, s3b: HTMLCanvasElement, s3c: HTMLCanvasElement;

	let triggers: ScrollTrigger[] = [];
	let rafIds: number[] = [];
	let timeouts: number[] = [];

	function handleMouseMove(e: MouseEvent) { mx = e.clientX; my = e.clientY; }
	function shake(el: HTMLElement, intensity = 20, dur = 0.5) { const s = Math.floor(dur / 0.016); for (let i = 0; i < s; i++) { const f = Math.pow(1 - i / s, 2); gsap.set(el, { x: (Math.random() - 0.5) * intensity * f, y: (Math.random() - 0.5) * intensity * f * 0.6, delay: i * 0.016 }); } gsap.set(el, { x: 0, y: 0, delay: s * 0.016 }); }

	function initImposters() {
		const cv = [s1a, s1b, s1c, s1d];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s1a.width = s1a.parentElement!.offsetWidth, h = s1a.height = s1a.parentElement!.offsetHeight;
		let t = 0, on = false, phase = 'fake';
		const fakePopups: { x: number; y: number; w: number; h: number; text: string; color: string; alpha: number; scale: number; life: number; rot: number; vx: number; vy: number }[] = [];
		const cracks: { pts: { x: number; y: number }[]; progress: number; width: number }[] = [];
		const shards: { x: number; y: number; vx: number; vy: number; w: number; h: number; rot: number; rv: number; life: number; color: string }[] = [];
		const FAKE_TEXTS = ['🚀 99.7% WIN RATE!!!', '💰 $50K IN ONE DAY!!!', '🔥 MY SECRET INDICATOR', 'JOIN NOW - $297/mo', '⚡ AI TRADING BOT', '📈 NEVER LOSE AGAIN', '🎯 100% ACCURACY', '💎 DIAMOND SIGNALS', 'FREE DISCORD (then $199)', '🤑 I QUIT MY JOB', 'COPY MY TRADES $$$', 'HOLY GRAIL SYSTEM', '10,000% GAINS!!!', 'RSI+MACD = MILLIONS', 'BUY MY COURSE $997', 'GURU MASTERCLASS', 'AUTOMATED PROFITS', 'GUARANTEED RETURNS', 'SECRET WALL ST HACK', 'NO EXPERIENCE NEEDED'];
		const FAKE_COLORS = ['#ff3366', '#ff6600', '#ffcc00', '#ff00ff', '#00ccff', '#ff4444'];
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s1a.width; h = s1a.height; }
		window.addEventListener('resize', resize);
		function spawnFake() { if (!on || phase !== 'fake') return; const txt = FAKE_TEXTS[Math.floor(Math.random() * FAKE_TEXTS.length)]; fakePopups.push({ x: Math.random() * (w - 300) + 50, y: Math.random() * (h - 100) + 50, w: 180 + Math.random() * 120, h: 35 + Math.random() * 25, text: txt, color: FAKE_COLORS[Math.floor(Math.random() * FAKE_COLORS.length)], alpha: 0, scale: 0, life: 1, rot: (Math.random() - 0.5) * 0.08, vx: (Math.random() - 0.5) * 0.3, vy: (Math.random() - 0.5) * 0.3 }); timeouts.push(window.setTimeout(spawnFake, 150 + Math.random() * 200)); }
		function smash() { phase = 'smash'; for (let i = 0; i < 20; i++) { const a = Math.random() * Math.PI * 2; const pts: { x: number; y: number }[] = []; let cx2 = w / 2, cy2 = h / 2; for (let j = 0; j < 12; j++) { cx2 += Math.cos(a + ((Math.random() - 0.5) * 0.5)) * 40; cy2 += Math.sin(a + ((Math.random() - 0.5) * 0.5)) * 40; pts.push({ x: cx2, y: cy2 }); } cracks.push({ pts, progress: 0, width: Math.random() * 3 + 1 }); } fakePopups.forEach(fp => { for (let i = 0; i < 6; i++) { const a = Math.random() * Math.PI * 2, spd = 5 + Math.random() * 10; shards.push({ x: fp.x + fp.w / 2, y: fp.y + fp.h / 2, vx: Math.cos(a) * spd, vy: Math.sin(a) * spd, w: fp.w / 3 + Math.random() * 20, h: fp.h / 2 + Math.random() * 10, rot: 0, rv: (Math.random() - 0.5) * 0.3, life: 1, color: fp.color }); } }); fakePopups.length = 0; }
		function draw() { if (!on) { rafIds.push(requestAnimationFrame(draw)); return; } t += 0.008; const [xa, xb, xc, xd] = ctx; xa.fillStyle = 'rgba(0,0,0,0.06)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xc.clearRect(0, 0, w, h); xd.clearRect(0, 0, w, h); if (phase === 'fake') { fakePopups.forEach(fp => { fp.alpha = Math.min(1, fp.alpha + 0.05); fp.scale = Math.min(1, fp.scale + 0.08); fp.x += fp.vx; fp.y += fp.vy; const a = fp.alpha * fp.life; xb.save(); xb.translate(fp.x + fp.w / 2, fp.y + fp.h / 2); xb.rotate(fp.rot); xb.scale(fp.scale, fp.scale); xb.fillStyle = `rgba(20,20,30,${a * 0.9})`; xb.strokeStyle = fp.color; xb.lineWidth = 2; xb.fillRect(-fp.w / 2, -fp.h / 2, fp.w, fp.h); xb.strokeRect(-fp.w / 2, -fp.h / 2, fp.w, fp.h); xb.fillStyle = fp.color; xb.font = `bold ${Math.floor(fp.h * 0.35)}px "JetBrains Mono"`; xb.textAlign = 'center'; xb.textBaseline = 'middle'; xb.fillText(fp.text, 0, 0); xb.textAlign = 'start'; xb.textBaseline = 'alphabetic'; xb.restore(); }); for (let i = 0; i < 3; i++) { const gx = Math.random() * w, gy = Math.random() * h; xa.fillStyle = 'rgba(255,0,100,0.01)'; xa.fillRect(gx, gy, Math.random() * 100, 2); } } if (phase === 'smash') { xd.globalCompositeOperation = 'lighter'; const coreGrd = xd.createRadialGradient(w / 2, h / 2, 0, w / 2, h / 2, 300 + t * 20); coreGrd.addColorStop(0, 'rgba(0,255,136,0.15)'); coreGrd.addColorStop(0.5, 'rgba(0,200,100,0.05)'); coreGrd.addColorStop(1, 'transparent'); xd.fillStyle = coreGrd; xd.fillRect(0, 0, w, h); xd.globalCompositeOperation = 'source-over'; cracks.forEach(cr => { if (cr.progress <= 0) return; const drawTo = Math.floor(cr.pts.length * Math.min(1, cr.progress)); xc.beginPath(); cr.pts.slice(0, drawTo).forEach((p, i) => { i === 0 ? xc.moveTo(p.x, p.y) : xc.lineTo(p.x, p.y); }); xc.strokeStyle = `rgba(0,255,136,${0.3 * Math.min(1, cr.progress)})`; xc.lineWidth = cr.width * 4; xc.shadowBlur = 20; xc.shadowColor = 'rgba(0,255,136,0.5)'; xc.stroke(); xc.beginPath(); cr.pts.slice(0, drawTo).forEach((p, i) => { i === 0 ? xc.moveTo(p.x, p.y) : xc.lineTo(p.x, p.y); }); xc.strokeStyle = `rgba(200,255,220,${0.6 * Math.min(1, cr.progress)})`; xc.lineWidth = cr.width; xc.stroke(); xc.shadowBlur = 0; }); shards.forEach(s => { s.x += s.vx; s.y += s.vy; s.vy += 0.08; s.rot += s.rv; s.life -= 0.008; if (s.life > 0) { xb.save(); xb.translate(s.x, s.y); xb.rotate(s.rot); xb.fillStyle = `rgba(30,30,40,${s.life * 0.6})`; xb.strokeStyle = s.color.replace(')', `,${s.life * 0.3})`).replace('rgb', 'rgba'); xb.lineWidth = 1; xb.fillRect(-s.w / 2, -s.h / 2, s.w, s.h); xb.restore(); } }); } rafIds.push(requestAnimationFrame(draw)); }
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s1', start: 'top 60%', onEnter: () => { on = true; spawnFake(); const tl = gsap.timeline(); tl.add(() => smash(), 3.5); tl.add(() => shake(document.getElementById('s1-content')!, 40, 0.8), 3.5); tl.add(() => { cracks.forEach((cr, i) => { gsap.to(cr, { progress: 1.2, duration: 0.6, delay: i * 0.02, ease: 'power3.out' }); }); }, 3.5); tl.to('#s1f', { opacity: 1, duration: 0.06 }, 3.8); tl.to('#s1f', { opacity: 0, duration: 2, ease: 'power3.out' }, 3.86); tl.set('#s1bk', { opacity: 1 }, 4.3); tl.to('#s1bk', { opacity: 0, duration: 0.03 }, 4.5); tl.to('#s1u0', { opacity: 1, duration: 0.4 }, 4.5); tl.to('#s1t', { opacity: 1, duration: 0.5 }, 4.8); tl.from('#s1t', { scale: 5, filter: 'blur(30px)', duration: 0.5, ease: 'power4.out' }, 4.8); tl.add(() => shake(document.getElementById('s1-content')!, 50, 0.8), 4.8); tl.to('#s1u', { opacity: 1, duration: 0.8 }, 5.5); tl.to('#s1v', { opacity: 1, duration: 1 }, 6.2); tl.to('#s1v2', { opacity: 1, duration: 1 }, 7); tl.to('#s1l', { opacity: 0.15, duration: 0.5 }, 5); } }); triggers.push(trigger);
	}

	function initOrderFlow() {
		const cv = [s2a, s2b];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s2a.width = s2a.parentElement!.offsetWidth, h = s2a.height = s2a.parentElement!.offsetHeight;
		let t = 0, on = false;
		let midPrice = 5892.50;
		const LEVELS = 40;
		const book: { bids: { price: number; size: number; flash: number }[]; asks: { price: number; size: number; flash: number }[] } = { bids: [], asks: [] };
		function initBook() { book.bids = []; book.asks = []; for (let i = 0; i < 20; i++) { book.bids.push({ price: midPrice - 0.25 * (i + 1), size: Math.floor(Math.random() * 300 + 20), flash: 0 }); book.asks.push({ price: midPrice + 0.25 * (i + 1), size: Math.floor(Math.random() * 300 + 20), flash: 0 }); } }
		initBook();
		function flowTick() { if (!on) return; const side = Math.random() > 0.5 ? book.bids : book.asks; const lvl = side[Math.floor(Math.random() * side.length)]; const delta = Math.floor((Math.random() - 0.4) * 80); lvl.size = Math.max(5, lvl.size + delta); if (Math.abs(delta) > 40) lvl.flash = 1; if (Math.random() > 0.95) { const blockSide = Math.random() > 0.5 ? book.bids : book.asks; const bl = blockSide[Math.floor(Math.random() * 3)]; bl.size += Math.floor(Math.random() * 500 + 200); bl.flash = 1; } timeouts.push(window.setTimeout(flowTick, 50 + Math.random() * 100)); }
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s2a.width; h = s2a.height; }
		window.addEventListener('resize', resize);
		function draw() { if (!on) { rafIds.push(requestAnimationFrame(draw)); return; } t += 0.008; const [xa, xb] = ctx; const midX = w / 2, rowH = h / LEVELS; xa.fillStyle = 'rgba(0,0,0,0.15)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); const maxSize = Math.max(...book.bids.map(b => b.size), ...book.asks.map(a => a.size)); book.bids.forEach((b, i) => { const y = h / 2 + i * rowH; const barW = (b.size / maxSize) * midX * 0.7; b.flash *= 0.93; xa.fillStyle = `rgba(0,255,136,${0.15 + b.flash * 0.4})`; xa.fillRect(midX - barW, y, barW, rowH - 1); xb.fillStyle = `rgba(0,255,136,${0.5 + b.flash * 0.5})`; xb.font = '10px "JetBrains Mono"'; xb.textAlign = 'right'; xb.fillText(b.size.toString(), midX - barW - 8, y + rowH * 0.7); xb.fillStyle = `rgba(0,255,136,${0.3})`; xb.textAlign = 'left'; xb.fillText(b.price.toFixed(2), midX - 65, y + rowH * 0.7); if (b.flash > 0.1) { xa.fillStyle = `rgba(0,255,136,${b.flash * 0.1})`; xa.fillRect(0, y, midX, rowH); } }); book.asks.forEach((a, i) => { const y = h / 2 - rowH - i * rowH; const barW = (a.size / maxSize) * midX * 0.7; a.flash *= 0.93; xa.fillStyle = `rgba(255,51,68,${0.15 + a.flash * 0.4})`; xa.fillRect(midX, y, barW, rowH - 1); xb.fillStyle = `rgba(255,51,68,${0.5 + a.flash * 0.5})`; xb.font = '10px "JetBrains Mono"'; xb.textAlign = 'left'; xb.fillText(a.size.toString(), midX + barW + 8, y + rowH * 0.7); xb.fillStyle = `rgba(255,51,68,${0.3})`; xb.textAlign = 'right'; xb.fillText(a.price.toFixed(2), midX + 65, y + rowH * 0.7); if (a.flash > 0.1) { xa.fillStyle = `rgba(255,51,68,${a.flash * 0.1})`; xa.fillRect(midX, y, w / 2, rowH); } }); xb.textAlign = 'start'; xb.strokeStyle = 'rgba(255,255,255,0.15)'; xb.lineWidth = 1; xb.beginPath(); xb.moveTo(midX, 0); xb.lineTo(midX, h); xb.stroke(); xb.fillStyle = 'rgba(255,255,255,0.6)'; xb.font = 'bold 14px "JetBrains Mono"'; xb.textAlign = 'center'; xb.fillText(`${midPrice.toFixed(2)}`, midX, h / 2 - 5); xb.fillStyle = 'rgba(255,255,255,0.2)'; xb.font = '8px "JetBrains Mono"'; xb.fillText('SPREAD: $0.25', midX, h / 2 + 12); xb.textAlign = 'start'; xb.setLineDash([3, 3]); xb.strokeStyle = 'rgba(255,255,255,0.08)'; xb.beginPath(); xb.moveTo(0, my); xb.lineTo(w, my); xb.stroke(); xb.setLineDash([]); rafIds.push(requestAnimationFrame(draw)); }
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s2', start: 'top 60%', onEnter: () => { on = true; flowTick(); const tl = gsap.timeline({ delay: 0.8 }); tl.to('#s2t', { opacity: 1, duration: 0.5 }, 1); tl.to('#s2u', { opacity: 1, duration: 0.8 }, 1.5); tl.to('#s2l', { opacity: 0.12, duration: 0.5 }, 1); } }); triggers.push(trigger);
	}

	function initSmartMoney() {
		const cv = [s3a, s3b, s3c];
		const ctx = cv.map(c => c.getContext('2d')!);
		let w = s3a.width = s3a.parentElement!.offsetWidth, h = s3a.height = s3a.parentElement!.offsetHeight;
		let t = 0, on = false, buildIdx = 0;
		const candles: { o: number; c: number; h: number; l: number; bull: boolean; phase: string }[] = [];
		let price = 5950;
		for (let i = 0; i < 8; i++) { const o = price; price -= Math.random() * 15 + 5; candles.push({ o, c: price, h: Math.max(o, price) + Math.random() * 5, l: Math.min(o, price) - Math.random() * 3, bull: false, phase: 'A' }); }
		const accLow = price, accHigh = price + 30;
		for (let i = 0; i < 20; i++) { const o = price; price += (Math.random() - 0.5) * 12; price = Math.max(accLow - 5, Math.min(accHigh + 5, price)); candles.push({ o, c: price, h: Math.max(o, price) + Math.random() * 3, l: Math.min(o, price) - Math.random() * 3, bull: price >= o, phase: 'B' }); }
		const o2 = price; price = accLow - 8; candles.push({ o: o2, c: price, h: o2 + 2, l: price - 3, bull: false, phase: 'C' }); price = accLow + 5; candles.push({ o: price - 13, c: price, h: price + 2, l: price - 15, bull: true, phase: 'C' });
		for (let i = 0; i < 12; i++) { const o = price; price += Math.random() * 15 + 5; candles.push({ o, c: price, h: price + Math.random() * 5, l: o - Math.random() * 3, bull: true, phase: 'D' }); }
		const arrows = [{ idx: 7, label: 'SELLING CLIMAX', color: '255,51,68' }, { idx: 15, label: 'ACCUMULATION', color: '74,168,255' }, { idx: 28, label: 'SPRING', color: '255,200,0' }, { idx: 32, label: 'MARKUP BEGINS', color: '0,255,136' }];
		function resize() { cv.forEach(c => { c.width = c.parentElement!.offsetWidth; c.height = c.parentElement!.offsetHeight; }); w = s3a.width; h = s3a.height; }
		window.addEventListener('resize', resize);
		function draw() { if (!on) { rafIds.push(requestAnimationFrame(draw)); return; } t += 0.008; const [xa, xb, xc] = ctx; xa.fillStyle = 'rgba(0,0,0,0.1)'; xa.fillRect(0, 0, w, h); xb.clearRect(0, 0, w, h); xc.clearRect(0, 0, w, h); const vis = candles.slice(0, buildIdx); if (!vis.length) { rafIds.push(requestAnimationFrame(draw)); return; } const ps = vis.flatMap(c => [c.h, c.l]); const minP = Math.min(...ps) - 5, maxP = Math.max(...ps) + 5, range = maxP - minP; const gap = w * 0.75 / candles.length, cW = Math.min(gap * 0.6, 10), offX = w * 0.12; if (buildIdx > 8) { const accTopY = h * 0.1 + (maxP - accHigh) / range * h * 0.7; const accBotY = h * 0.1 + (maxP - accLow) / range * h * 0.7; xb.fillStyle = `rgba(74,168,255,${0.04 + Math.sin(t * 2) * 0.01})`; xb.fillRect(offX + 8 * gap, accTopY, 20 * gap, accBotY - accTopY); xb.strokeStyle = 'rgba(74,168,255,0.15)'; xb.setLineDash([4, 4]); xb.lineWidth = 1; xb.strokeRect(offX + 8 * gap, accTopY, 20 * gap, accBotY - accTopY); xb.setLineDash([]); xb.fillStyle = 'rgba(74,168,255,0.12)'; xb.font = '9px Orbitron'; xb.fillText('INSTITUTIONAL ACCUMULATION ZONE', offX + 8 * gap + 10, accTopY - 6); } vis.forEach((c, idx) => { const i = candles.indexOf(c); const x = offX + i * gap + gap / 2; const oY = h * 0.1 + (maxP - c.o) / range * h * 0.7; const cY = h * 0.1 + (maxP - c.c) / range * h * 0.7; const hY = h * 0.1 + (maxP - c.h) / range * h * 0.7; const lY = h * 0.1 + (maxP - c.l) / range * h * 0.7; const clr = c.bull ? '0,255,136' : '255,51,68'; xb.strokeStyle = `rgba(${clr},0.5)`; xb.lineWidth = 1; xb.beginPath(); xb.moveTo(x, hY); xb.lineTo(x, lY); xb.stroke(); xb.fillStyle = `rgba(${clr},0.7)`; xb.fillRect(x - cW / 2, Math.min(oY, cY), cW, Math.abs(cY - oY) || 1.5); if (c.phase === 'D') { xb.shadowBlur = 12; xb.shadowColor = 'rgba(0,255,136,0.4)'; xb.fillRect(x - cW / 2, Math.min(oY, cY), cW, Math.abs(cY - oY) || 1.5); xb.shadowBlur = 0; } }); arrows.forEach(ar => { if (buildIdx <= ar.idx) return; const c = candles[ar.idx]; const x = offX + ar.idx * gap + gap / 2; const y = h * 0.1 + (maxP - c.l) / range * h * 0.7 + 25; xc.fillStyle = `rgba(${ar.color},0.5)`; xc.font = 'bold 8px "JetBrains Mono"'; xc.textAlign = 'center'; xc.fillText(ar.label, x, y); xc.beginPath(); xc.moveTo(x, y - 18); xc.lineTo(x, y - 8); xc.strokeStyle = `rgba(${ar.color},0.4)`; xc.lineWidth = 1.5; xc.stroke(); xc.textAlign = 'start'; }); rafIds.push(requestAnimationFrame(draw)); }
		rafIds.push(requestAnimationFrame(draw));
		const trigger = ScrollTrigger.create({ trigger: '#s3', start: 'top 60%', onEnter: () => { on = true; const total = candles.length; candles.forEach((c, i) => { const d = i < 8 ? i * 0.08 : i < 28 ? 8 * 0.08 + (i - 8) * 0.06 : 28 * 0.06 + (i - 28) * 0.04; timeouts.push(window.setTimeout(() => { buildIdx = i + 1; }, d * 1000)); }); const endTime = 28 * 0.06 + 12 * 0.04 + 0.5; const tl = gsap.timeline(); tl.to('#s3f', { opacity: 0.3, duration: 0.05 }, endTime); tl.to('#s3f', { opacity: 0, duration: 1 }, endTime + 0.05); tl.to('#s3t', { opacity: 1, duration: 0.5 }, endTime); tl.from('#s3t', { scale: 2, filter: 'blur(15px)', duration: 0.5, ease: 'power4.out' }, endTime); tl.add(() => shake(document.getElementById('s3-content')!, 15, 0.3), endTime); tl.to('#s3u', { opacity: 1, duration: 1 }, endTime + 0.5); tl.to('#s3l', { opacity: 0.12, duration: 0.5 }, endTime + 0.3); } }); triggers.push(trigger);
	}

	onMount(() => { mx = window.innerWidth / 2; my = window.innerHeight / 2; initImposters(); initOrderFlow(); initSmartMoney(); });
	onDestroy(() => { triggers.forEach(t => t.kill()); rafIds.forEach(id => cancelAnimationFrame(id)); timeouts.forEach(id => clearTimeout(id)); });
</script>

<svelte:window on:mousemove={handleMouseMove} />

<section class="hero" id="s1">
	<canvas bind:this={s1a}></canvas>
	<canvas bind:this={s1b} class="c2"></canvas>
	<canvas bind:this={s1c} class="c3"></canvas>
	<canvas bind:this={s1d} class="c4"></canvas>
	<div class="flash" id="s1f"></div>
	<div class="bk" id="s1bk"></div>
	<div class="ct" id="s1-content">
		<p class="t3" id="s1u0">WHILE THEY SELL DREAMS</p>
		<h1 class="t1" id="s1t">REVOLUTION<br>TRADING PROS</h1>
		<p class="t2" id="s1u">WE TEACH THE REAL GAME</p>
		<p class="t4" id="s1v">PURE PRICE ACTION // INSTITUTIONAL STRATEGIES<br>LEARNED ON WALL STREET // NOT FROM A YOUTUBE AD</p>
		<p class="t5" id="s1v2">MENTORED BY MARK McGOLDRICK — GOLDMAN SACHS</p>
	</div>
	<span class="lbl" id="s1l">01 — THE REVOLUTION</span>
</section>
<div class="sp"></div>

<section class="hero" id="s2">
	<canvas bind:this={s2a}></canvas>
	<canvas bind:this={s2b} class="c2"></canvas>
	<div class="flash" id="s2f"></div>
	<div class="ct" id="s2-content">
		<h1 class="t1" id="s2t">ORDER FLOW</h1>
		<p class="t2" id="s2u">SEE WHERE THE REAL MONEY IS MOVING</p>
	</div>
	<span class="lbl" id="s2l">02 — THE BOOK</span>
</section>
<div class="sp"></div>

<section class="hero" id="s3">
	<canvas bind:this={s3a}></canvas>
	<canvas bind:this={s3b} class="c2"></canvas>
	<canvas bind:this={s3c} class="c3"></canvas>
	<div class="flash" id="s3f"></div>
	<div class="ct" id="s3-content">
		<h1 class="t1" id="s3t">SMART MONEY</h1>
		<p class="t2" id="s3u">FOLLOW THE INSTITUTIONS. NOT THE CROWD.</p>
	</div>
	<span class="lbl" id="s3l">03 — ACCUMULATION</span>
</section>

<style>
	:global(*) { margin: 0; padding: 0; box-sizing: border-box; }
	:global(html) { overflow-x: hidden; background: #000; }
	.hero { position: relative; width: 100%; height: 100vh; overflow: hidden; display: flex; align-items: center; justify-content: center; color: #fff; font-family: 'JetBrains Mono', monospace; }
	.hero canvas { position: absolute; inset: 0; width: 100%; height: 100%; z-index: 1; }
	.c2 { z-index: 2; } .c3 { z-index: 3; } .c4 { z-index: 4; }
	.flash { position: absolute; inset: 0; z-index: 30; pointer-events: none; opacity: 0; }
	.bk { position: absolute; inset: 0; z-index: 35; pointer-events: none; background: #000; opacity: 0; }
	.ct { position: relative; z-index: 20; text-align: center; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%; pointer-events: none; }
	.lbl { position: absolute; top: 25px; left: 30px; z-index: 60; font-size: 0.38rem; letter-spacing: 0.7em; text-transform: uppercase; opacity: 0.15; }
	.t1 { font-family: 'Anton', sans-serif; font-size: clamp(3.5rem, 13vw, 12rem); line-height: 0.85; letter-spacing: -0.01em; opacity: 0; }
	.t2 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.4rem, 1vw, 0.65rem); letter-spacing: 0.7em; margin-top: 15px; opacity: 0; }
	.t3 { font-family: 'Orbitron', sans-serif; font-size: clamp(0.35rem, 0.8vw, 0.5rem); letter-spacing: 1em; margin-bottom: 10px; opacity: 0; }
	.t4 { font-size: 0.45rem; letter-spacing: 0.35em; margin-top: 22px; max-width: 700px; line-height: 1.8; opacity: 0; }
	.t5 { font-size: 0.4rem; letter-spacing: 0.2em; margin-top: 15px; opacity: 0; }
	.sp { height: 2px; background: #000; }
	#s1 { background: #000; } #s1f { background: radial-gradient(circle, rgba(0,255,136,0.9), rgba(0,100,50,0.4) 40%, transparent 65%); }
	#s1t { background: linear-gradient(180deg, #fff, #00ff88, #006633); -webkit-background-clip: text; background-clip: text; color: transparent; font-size: clamp(2.2rem, 7vw, 6.5rem); }
	#s1u { color: rgba(0,255,136,0.6); } #s1u0 { color: rgba(0,255,136,0.7); } #s1v { color: rgba(0,255,136,0.35); } #s1v2 { color: rgba(0,255,136,0.25); }
	#s1l { color: #00ff88; }
	#s2 { background: #000; } #s2f { background: rgba(0,255,136,0.3); }
	#s2t { color: #fff; text-shadow: 0 0 60px rgba(0,255,136,0.3); }
	#s2u { color: rgba(0,255,136,0.5); } #s2l { color: #00ff88; }
	#s3 { background: #000; } #s3f { background: rgba(100,180,255,0.3); }
	#s3t { color: #4aa8ff; text-shadow: 0 0 80px rgba(74,168,255,0.4); }
	#s3u { color: rgba(74,168,255,0.5); } #s3l { color: #4aa8ff; }
</style>
