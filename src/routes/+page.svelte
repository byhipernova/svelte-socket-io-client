<script lang="ts">
	import { onDestroy, onMount } from 'svelte';
	import { io, Socket } from 'socket.io-client';
	import { Trash2 } from 'lucide-svelte';

	let socket = $state<Socket | null>(null);
	let url = $state('');
	let logs = $state<
		{
			message: string;
			event: string;
			sender: 'client' | 'server';
		}[]
	>([]);
	let message = $state('');
	let messageType = $state('message');
	let isConnected = $state<boolean>(false);

	const desktopTypes: number[] = [103, 105, 107, 109, 113];
	const mobileTypes: number[] = [104, 106, 108, 110, 111, 112, 114];

	onMount(() => {
		socket?.disconnect();
	});

	onDestroy(() => {
		socket?.disconnect();
	});

	const connect = () => {
		if (socket) socket.disconnect();
		socket = io(url);
		socket.on('connect', () => (isConnected = true));
		socket.on('disconnect', () => (isConnected = false));
		socket.onAny((eventName, ...args) => {
			for (const arg of args) {
				logs = [
					...logs,
					{
						message: typeof arg === 'string' ? arg : JSON.stringify(arg),
						event: eventName,
						sender: 'server'
					}
				];
			}
		});
	};

	const sendMessage = (messageType: string, message: string | Record<string, any>) => {
		if (socket === null) return;
		socket.emit(messageType, message);
		const data = typeof message === 'string' ? message : JSON.stringify(message);
		logs = [...logs, { message: data, event: messageType, sender: 'client' }];
		message = '';
	};
</script>

<div class="flex h-screen w-screen gap-4 bg-gray-100 p-4">
	<div class="w-2/6 bg-white p-4 shadow">
		<input
			bind:value={url}
			class="mb-2 w-full rounded border p-2"
			placeholder="Socket URL"
			disabled={isConnected}
		/>
		{#if isConnected}
			<button
				onclick={() => socket?.disconnect()}
				class="w-full rounded bg-red-500 p-2 text-white">Disconnect</button
			>
		{:else}
			<button onclick={connect} class="w-full rounded bg-blue-400 p-2 text-white"
				>Connect</button
			>
		{/if}

		<div class="mt-4">
			<h2 class="pb-2 font-bold">Desktop</h2>
			<div class="flex flex-wrap gap-2">
				{#each desktopTypes as type}
					<button
						class=" rounded bg-blue-400 px-4 py-2 text-white"
						onclick={() => sendMessage('message', { type })}>{type}</button
					>
				{/each}
			</div>
		</div>

		<div class="mt-4">
			<h2 class="pb-2 font-bold">Mobile</h2>
			<div class=" flex flex-wrap gap-2">
				{#each mobileTypes as type}
					<button
						class=" rounded bg-green-400 px-4 py-2 text-white"
						onclick={() => sendMessage('message', { type })}>{type}</button
					>
				{/each}
			</div>
		</div>

		<div class="mt-4">
			<h2 class="font-bold">Send Message</h2>
			<input
				bind:value={messageType}
				class="mb-2 w-full rounded border p-2"
				placeholder="Message Type"
			/>
			<textarea
				bind:value={message}
				class="h-24 w-full rounded border p-2"
				placeholder={`{\"key\": \"value\"}`}
			></textarea>
			<button
				class="mt-2 w-full rounded bg-green-500 p-2 text-white"
				onclick={() => sendMessage(messageType, message)}>Send</button
			>
		</div>
	</div>

	<div class="w-4/6 overflow-auto bg-white p-4 shadow">
		<div class="flex content-between justify-between gap-2">
			<h2 class="font-bold">Events</h2>
			<button class="" onclick={() => (logs = [])}><Trash2 size="20" /></button>
		</div>
		<div class="mt-2 h-full overflow-y-auto border bg-gray-700 p-2 text-sm">
			{#each logs as log}
				<div class="mt-2 bg-white shadow-sm">
					<div
						class="inline-block min-w-24 px-2 py-2 text-left text-white {log.sender ===
						'client'
							? 'bg-green-400'
							: 'bg-blue-400 '}"
					>
						{log.event}
					</div>
					<div class="inline-block px-4 py-2 text-gray-700">{log.message}</div>
				</div>
			{/each}
		</div>
	</div>
</div>
