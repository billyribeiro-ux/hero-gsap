<script lang="ts">
	import { page } from '$app/state';
	import { goto } from '$app/navigation';

	const sets = [
		{ id: 'ten-universes-1', name: 'Ten Universes (1-3)', path: '/universes/1' },
		{ id: 'ten-universes-2', name: 'Ten Universes (4-6)', path: '/universes/2' },
		{ id: 'ten-universes-3', name: 'Ten Universes (7-9)', path: '/universes/3' },
		{ id: 'ten-universes-4', name: 'Ten Universes (10)', path: '/universes/4' },
		{ id: 'trading-1', name: 'Trading (1-3)', path: '/trading/1' },
		{ id: 'trading-2', name: 'Trading (4-6)', path: '/trading/2' },
		{ id: 'trading-3', name: 'Trading (7-9)', path: '/trading/3' },
		{ id: 'trading-4', name: 'Trading (10)', path: '/trading/4' },
		{ id: 'absolute-1', name: 'Absolute Max (1-3)', path: '/absolute/1' },
		{ id: 'absolute-2', name: 'Absolute Max (4-5)', path: '/absolute/2' },
		{ id: 'magnum-opus-1', name: 'Magnum Opus (1-3)', path: '/magnum-opus/1' },
		{ id: 'magnum-opus-2', name: 'Magnum Opus (4-6)', path: '/magnum-opus/2' },
		{ id: 'magnum-opus-3', name: 'Magnum Opus (7-9)', path: '/magnum-opus/3' },
		{ id: 'magnum-opus-4', name: 'Magnum Opus (10)', path: '/magnum-opus/4' }
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
