<script lang="ts">
	import AnimationNav from '$lib/components/AnimationNav.svelte';
	import TenUniverses from '$lib/components/TenUniverses.svelte';
</script>

<!-- Only show first 5 via CSS - for performance we use the full component but hide last 5 -->
<div class="five-only">
	<TenUniverses />
</div>
<AnimationNav />

<style>
	.five-only :global(#s6),
	.five-only :global(#s7),
	.five-only :global(#s8),
	.five-only :global(#s9),
	.five-only :global(#s10),
	.five-only :global(#s6 + .divider),
	.five-only :global(#s7 + .divider),
	.five-only :global(#s8 + .divider),
	.five-only :global(#s9 + .divider) {
		display: none !important;
	}
</style>
