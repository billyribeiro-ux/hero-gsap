<script lang="ts">
	import AnimationNav from '$lib/components/AnimationNav.svelte';
	import TradingUniverses from '$lib/components/TradingUniverses.svelte';
</script>

<!-- Only show first 5 via CSS -->
<div class="five-only">
	<TradingUniverses />
</div>
<AnimationNav />

<style>
	.five-only :global(#h6),
	.five-only :global(#h7),
	.five-only :global(#h8),
	.five-only :global(#h9),
	.five-only :global(#h10),
	.five-only :global(#h6 + .sp),
	.five-only :global(#h7 + .sp),
	.five-only :global(#h8 + .sp),
	.five-only :global(#h9 + .sp) {
		display: none !important;
	}
</style>
