<script lang="ts">
	import { page } from '$app/stores';
	import { supabase } from '$lib/glue/supabaseClient';
	import IconAdd from '$lib/icons/glue/IconAdd.svelte';
	import { onMount } from 'svelte';
	import TextInput from './glue/TextInput.svelte';

	type TChannelsResponse = Awaited<ReturnType<typeof getChannels>>;

	let channels: TChannelsResponse['data'] = [];
	let newChannelName: string = '';
	let isCreateChannelModalOpen: boolean = false;

	const getChannels = async () => {
		return await supabase.from('channels').select();
	};

	const createChannel = async () => {
		if ($page?.data?.session) {
			const { data } = await supabase
				.from('channels')
				.insert({
					created_by: $page?.data?.session?.user?.id,
					slug: newChannelName
				})
				.select()
				.single();

			if (data) {
				newChannelName = '';
				isCreateChannelModalOpen = false;
				channels = channels ? [...channels, data] : [data];
			}
		}
	};

	onMount(async () => {
		channels = (await getChannels())?.data;
	});
</script>

{#if $page?.data?.session}
	<div class="space-y-2">
		<h2 class="ml-2 text-xl font-bold">Channels</h2>
		<div>
			{#if Array?.isArray(channels)}
				{#each channels as channel (channel?.id)}
					<a href="/channel/{channel?.id}">
						<div>
							<button class="btn-ghost btn-sm btn">
								# {channel?.slug}
							</button>
						</div>
					</a>
				{/each}
			{/if}
			<div>
				<button
					class="btn-ghost btn-sm btn gap-1 pl-2"
					on:click={() => {
						isCreateChannelModalOpen = true;
					}}><span class="text-lg"><IconAdd /></span> Add channel</button
				>
			</div>
		</div>
	</div>
{:else}
	<p>sign in to view channels</p>
{/if}

<!-- modal: create channel -->
<input type="checkbox" id="modal-create-channel" class="modal-toggle" />
<label
	for="modal-create-channel"
	class="modal cursor-pointer {isCreateChannelModalOpen && 'modal-open'}"
>
	<label class="modal-box relative" for="">
		<form class="space-y-4" on:submit|preventDefault={createChannel}>
			<h3 class="text-lg font-bold">Add channel</h3>
			<TextInput bind:value={newChannelName} label="Channel name" />
			<div class="flex justify-end">
				<button type="submit" class="btn-primary btn-sm btn">Add channel</button>
			</div>
		</form>
	</label>
</label>
