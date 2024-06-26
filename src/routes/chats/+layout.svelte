<script>
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';
	import { onDestroy, onMount } from 'svelte';
	import { getLoggedInUser } from '../../database/auth';
	import { subscribeUserChats, createChatRoom } from '../../database/chats';
	import ChatTile from '../../components/ChatTile.svelte';
	import Spinner from '../../components/shared/Spinner.svelte';

	let user = null;

	let chats = null;
	let createChatBtnDisabled = true;
	// undefined if no chat is clicked
	$: selectedChat = $page.params.chatId;

	let unsubscribeUserChats = () => {};

	onMount(() => {
		user = getLoggedInUser();
		if (user === null) {
			// redirect a user if not logged in
			goto('/login');
		} else {
			createChatBtnDisabled = false;
			unsubscribeUserChats = subscribeUserChats(user.id, (newChatsData) => {
				chats = newChatsData;
			});
		}
	});

	onDestroy(() => {
		unsubscribeUserChats();
	});

	async function handleCreateChat() {
		createChatBtnDisabled = true;
		const newChatId = await createChatRoom(user.id);
		goto(`/chats/${newChatId}`);
		createChatBtnDisabled = false;
	}
</script>

<svelte:head>
	<title>Chat Rooms</title>
</svelte:head>

{#if user !== null}
	<div class="chats-container">
		<aside>
			<div class="actions">
				<button
					on:click={handleCreateChat}
					disabled={createChatBtnDisabled}
					class="btn btn-primary create-chatroom-btn"
				>
					New chat
				</button>
			</div>
			{#if chats !== null}
				{#if chats.length === 0}
					<p>No chats available</p>
				{:else}
					<div class="chat-list-container">
						{#each chats as chat}
							<ChatTile
								chatId={chat.id}
								chatName={chat.name}
								lastMessage={chat.lastMessage}
								lastMessageTimestamp={chat.lastMessageTimestamp}
								isSelected={chat.id === selectedChat}
							/>
						{/each}
					</div>
				{/if}
			{:else}
				<div class="spinner-container">
					<Spinner />
				</div>
			{/if}
		</aside>
		<main>
			<slot />
		</main>
	</div>
{/if}

<style>
	.spinner-container {
		width: 100%;
		display: flex;
		justify-content: center;
		padding: 24px 0;
	}

	.chats-container {
		display: grid;
		gap: 24px;
		grid-template-columns: repeat(12, minmax(0, 1fr));
		min-height: 100%;
		padding: 10px 24px 0;
	}

	.chats-container aside {
		grid-column: span 3 / span 3;
		padding-top: 6px;
	}

	.chats-container main {
		grid-column: span 9 / span 9;
		padding-top: 6px;
	}

	.chat-list-container {
		display: flex;
		flex-direction: column;
		gap: 16px;
		margin-top: 16px;
		padding-right: 4px;
		height: calc(100vh - 80.875px - 6px - 34px - 16px - 59px);
	}

	main {
		height: calc(100vh - 80.875px - 6px - 59px);
	}

	.chat-list-container {
		overflow-y: scroll;
	}

	.actions {
		display: flex;
		flex-direction: row;
		gap: 8px;
	}

	.actions button {
		font-weight: bold;
		font-size: 1em;
		border-radius: 4px;
		padding: 6px;
		cursor: pointer;
		width: 100%;
	}

	.create-chatroom-btn:disabled {
		cursor: auto;
	}
</style>
