<script lang="ts">
	import { page } from '$app/state';
	import { goto } from '$app/navigation';

	const sets = [
		{ id: 'ten-universes-1', name: 'Ten Universes (1-5)', path: '/universes/1' },
		{ id: 'ten-universes-2', name: 'Ten Universes (6-10)', path: '/universes/2' },
		{ id: 'trading-1', name: 'Trading Universes (1-5)', path: '/trading/1' },
		{ id: 'trading-2', name: 'Trading Universes (6-10)', path: '/trading/2' },
		{ id: 'absolute-max', name: 'Absolute Maximum', path: '/absolute' }
	];

	let currentIndex = $derived(sets.findIndex(s => s.path === page.url.pathname) ?? -1);

	function prev() {
		if (currentIndex > 0) goto(sets[currentIndex - 1].path);
	}

	function next() {
		if (currentIndex < sets.length - 1) goto(sets[currentIndex + 1].path);
	}
</script>

<nav class="nav-bar">
	<button class="nav-btn" onclick={prev} disabled={currentIndex <= 0}>← PREV</button>
	<div class="dots">
		{#each sets as s, i}
			<a href={s.path} class="dot" class:active={i === currentIndex} title={s.name}></a>
		{/each}
	</div>
	<button class="nav-btn" onclick={next} disabled={currentIndex >= sets.length - 1}>NEXT →</button>
</nav>

<style>
	.nav-bar {
		position: fixed;
		bottom: 20px;
		left: 50%;
		transform: translateX(-50%);
		display: flex;
		align-items: center;
		gap: 20px;
		z-index: 1000;
		background: rgba(0,0,0,0.8);
		padding: 12px 24px;
		border-radius: 30px;
		backdrop-filter: blur(10px);
		border: 1px solid rgba(255,255,255,0.1);
	}

	.nav-btn {
		background: transparent;
		border: 1px solid rgba(255,255,255,0.3);
		color: #fff;
		padding: 8px 16px;
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.7rem;
		letter-spacing: 0.1em;
		cursor: pointer;
		border-radius: 4px;
		transition: all 0.3s ease;
	}

	.nav-btn:hover:not(:disabled) {
		background: rgba(255,255,255,0.1);
		border-color: rgba(255,255,255,0.5);
	}

	.nav-btn:disabled {
		opacity: 0.3;
		cursor: not-allowed;
	}

	.dots {
		display: flex;
		gap: 8px;
	}

	.dot {
		width: 10px;
		height: 10px;
		border-radius: 50%;
		background: rgba(255,255,255,0.2);
		transition: all 0.3s ease;
	}

	.dot:hover {
		background: rgba(255,255,255,0.4);
	}

	.dot.active {
		background: #00ff88;
		box-shadow: 0 0 10px rgba(0,255,136,0.5);
	}
</style>
