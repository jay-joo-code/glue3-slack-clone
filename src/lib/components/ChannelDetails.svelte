<script lang="ts">
	import { page } from '$app/stores';
	import { supabase } from '$lib/glue/supabaseClient';
	import type { RealtimeChannel } from '@supabase/realtime-js';
	import { formatDistanceToNowStrict } from 'date-fns';
	import TextInput from './glue/TextInput.svelte';

	type TMessagesResponse = Awaited<ReturnType<typeof fetchMessages>>;

	let messages: TMessagesResponse['data'] = [];
	let message: string = '';
	let realtimeChannel: RealtimeChannel | null = null;

	const fetchMessages = async () =>
		await supabase.from('messages').select('*, users(*)').eq('channel_id', $page?.data?.channelId);
	const getMessages = async () => (messages = (await fetchMessages())?.data);

	const sendMessage = async () => {
		if ($page?.data?.session) {
			try {
				const { data, error } = await supabase
					.from('messages')
					.insert({
						user_id: $page?.data?.session?.user?.id,
						message,
						channel_id: $page?.data?.channelId
					})
					.select('*, users(*)')
					.single();
				if (error) throw error;
				if (realtimeChannel) {
					realtimeChannel.send({
						type: 'broadcast',
						event: 'new-message',
						payload: data
					});
				}
				message = '';
			} catch (error) {}
		}
	};

	const subscribe = () => {
		realtimeChannel = supabase.channel($page?.data?.channelId, {
			config: {
				broadcast: {
					self: true
				}
			}
		});
		realtimeChannel
			.on('broadcast', { event: 'new-message' }, ({ payload }) => {
				if (payload) {
					messages = messages ? [...messages, payload] : [payload];
				}
			})
			.subscribe();
	};

	$: if ($page?.data?.channelId) {
		getMessages();
		subscribe();
	}
</script>

{#if $page?.data?.session}
	<div class="relative h-[76vh]">
		<!-- messages list -->
		{#if messages}
			<div>
				{#each messages as message (message?.id)}
					<div class="chat chat-start">
						<div class="chat-header mb-0.5">
							{#if !Array.isArray(message?.users)}
								{message?.users?.username}
							{/if}
							<time class="text-xs opacity-50"
								>{formatDistanceToNowStrict(new Date(message?.inserted_at))} ago</time
							>
						</div>
						<div class="chat-bubble">{message?.message}</div>
					</div>
				{/each}
			</div>
		{/if}

		<!-- text input -->
		<div class="fixed bottom-4 w-[90vw] bg-base-100/60 p-2 md:absolute md:bottom-6 md:w-full">
			<form on:submit|preventDefault={sendMessage}>
				<TextInput
					class="rounded-full border-base-content/60"
					bind:value={message}
					placeholder="Enter chat here"
				/>
			</form>
		</div>
	</div>
{:else}
	<p>sign in to view channel details</p>
{/if}
