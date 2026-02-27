<script lang="ts">
	import AnimationNav from '$lib/components/AnimationNav.svelte';
	import TenUniverses from '$lib/components/TenUniverses.svelte';
</script>

<!-- Only show last 5 via CSS -->
<div class="five-only">
	<TenUniverses />
</div>
<AnimationNav />

<style>
	.five-only :global(#s1),
	.five-only :global(#s2),
	.five-only :global(#s3),
	.five-only :global(#s4),
	.five-only :global(#s5),
	.five-only :global(#s1 + .divider),
	.five-only :global(#s2 + .divider),
	.five-only :global(#s3 + .divider),
	.five-only :global(#s4 + .divider) {
		display: none !important;
	}
</style>
