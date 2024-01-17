<script lang="ts">
	import { onDestroy, onMount, tick } from 'svelte';
	import { getModalStore, getToastStore, type ModalSettings } from '@skeletonlabs/skeleton';
	import hljs from 'highlight.js';
	import { XMark } from '@inqling/svelte-icons/heroicon-24-solid';
	import { Trash, Cog6Tooth, Share } from '@inqling/svelte-icons/heroicon-24-outline';
	import type { PageData } from './$types';
	import { goto } from '$app/navigation';
	import { chatStore, isLoadingAnswerStore, settingsStore } from '$misc/stores';
	import Toolbar from '$lib/Toolbar.svelte';
	import ChatInput from '$lib/ChatInput.svelte';
	import Chat from '$lib/Chat.svelte';
	import HintMessage from '$lib/HintMessage.svelte';
	import TokenCost from '$lib/TokenCost.svelte';
	import { countTokens, estimateChatCost } from '$misc/openai';
	import {
		canSuggestTitle,
		createNewChat,
		showModalComponent,
		showToast,
		suggestChatTitle,
		track,
		type ChatMessage
	} from '$misc/shared';
	import snarkdown from 'snarkdown';

	const modalStore = getModalStore();
	const toastStore = getToastStore();

	export let data: PageData;

	$: ({ slug } = data);
	$: chat = $chatStore[slug];
	$: cost = chat ? estimateChatCost(chat) : null;
	$: hasContext = chat?.contextMessage?.content?.length || false;
	$: hasStopSequence = chat?.settings.stop?.length || false;

	let chatInput: ChatInput;

	onMount(async () => {
		await highlightCode();
	});

	const unsubscribeChatStore = chatStore.subscribe(async () => {
		await highlightCode();
	});

	const unsubscribeisLoadingAnswerStore = isLoadingAnswerStore.subscribe(async () => {
		await highlightCode();
	});

	onDestroy(() => {
		unsubscribeChatStore();
		unsubscribeisLoadingAnswerStore();
	});

	async function highlightCode() {
		await tick();
		hljs.highlightAll();
	}

	function showConfirmDeleteModal() {
		const modal: ModalSettings = {
			type: 'confirm',
			title: 'Please confirm',
			body: 'Are you sure you want to delete this chat?',
			response: (result: boolean) => {
				if (result) {
					deleteChat();
				}
			}
		};
		modalStore.trigger(modal);
	}

	const handleChatShared = (savedSlug: boolean) => {
		// sharing might have updated the slug of this chat
		// so we are either already on that page, or we go there
		if (savedSlug) {
			goto(`/${savedSlug}`);
		}
	};

	function deleteChat(dontTrack = false) {
		if (!dontTrack) {
			track('deleteChat');
		}
		chatStore.deleteChat(slug);
		goto('/');
	}

	async function handleCloseChat() {
		// untouched => discard
		if (chat.title === slug && chat.messages.length === 0) {
			showToast(toastStore, 'Empty chat was discarded automatically', 'secondary');
			deleteChat(true);
		}

		// already has a title
		if (chat.title !== slug || !canSuggestTitle(chat)) {
			goto('/');
			return;
		}

		// has no title
		if ($settingsStore.useTitleSuggestions) {
			if ($settingsStore.openAiApiKey) {
				const title = await suggestChatTitle(chat, $settingsStore.openAiApiKey);
				chatStore.updateChat(slug, { title });
				showToast(toastStore, `Chat title set to: '${title}'`, 'secondary');
			}
			goto('/');
		} else {
			showModalComponent(modalStore, 'SuggestTitleModal', { slug }, () => {
				setTimeout(() => goto('/'), 0);
			});
		}
	}

	function handleRenameChat(event: CustomEvent<string>) {
		chatStore.updateChat(slug, { title: event.detail });
	}

	function handleEditMessage(event: CustomEvent<ChatMessage>) {
		chatInput.editMessage(event.detail);
	}
</script>

{#if chat}
	<Toolbar
		{slug}
		title={chat.title}
		on:closeChat={handleCloseChat}
		on:renameChat={handleRenameChat}
	>
		<svelte:fragment slot="actions">
			<!-- Delete -->
			<button class="btn btn-sm variant-ghost-error" on:click={showConfirmDeleteModal}>
				<Trash class="w-6 h-6" />
			</button>
		</svelte:fragment>
	</Toolbar>

	<Chat {slug} on:editMessage={handleEditMessage}>
		<svelte:fragment slot="additional-content-top">
			<!-- Language hint -->
			{#if !$settingsStore.hideLanguageHint}
				<HintMessage title="Did you know?" variantClass="variant-ghost-surface">
					<p>
						All data is stored on your computer, which ensures patient confidentiality.
					</p>
					<svelte:fragment slot="actions">
						<button class="btn btn-sm" on:click={() => ($settingsStore.hideLanguageHint = true)}>
							<XMark class="w-6 h-6" />
						</button>
					</svelte:fragment>
				</HintMessage>
			{/if}

		</svelte:fragment>
	</Chat>

	<ChatInput {slug} chatCost=0 bind:this={chatInput} />
{/if}
