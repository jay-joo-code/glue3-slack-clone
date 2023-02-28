<script>
	import { page } from '$app/stores';
</script>

{#if $page?.data?.session}
	<div>
		<p>channel details</p>
	</div>
{:else}
	<p>sign in to view channel details</p>
{/if}
