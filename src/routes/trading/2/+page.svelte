<script lang="ts">
	import AnimationNav from '$lib/components/AnimationNav.svelte';
	import TradingUniverses from '$lib/components/TradingUniverses.svelte';
</script>

<!-- Only show last 5 via CSS -->
<div class="five-only">
	<TradingUniverses />
</div>
<AnimationNav />

<style>
	.five-only :global(#h1),
	.five-only :global(#h2),
	.five-only :global(#h3),
	.five-only :global(#h4),
	.five-only :global(#h5),
	.five-only :global(#h1 + .sp),
	.five-only :global(#h2 + .sp),
	.five-only :global(#h3 + .sp),
	.five-only :global(#h4 + .sp) {
		display: none !important;
	}
</style>
