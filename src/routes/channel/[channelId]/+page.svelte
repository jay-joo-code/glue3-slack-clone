<script>
	import { page } from '$app/stores';
	import ChannelDetails from '$lib/components/ChannelDetails.svelte';
	import Channels from '$lib/components/Channels.svelte';
	import Aside from '$lib/components/glue/Aside.svelte';
	import Main from '$lib/components/glue/Main.svelte';
	import PageContainer from '$lib/components/glue/PageContainer.svelte';
</script>

<PageContainer title="Home" layout="aside-main">
	<Aside>
		<Channels />
	</Aside>
	<Main>
		<ChannelDetails />
	</Main>
</PageContainer>
