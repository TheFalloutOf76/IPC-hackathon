<script lang="ts">
	import { onMount } from 'svelte';
	import TimeAgo from 'javascript-time-ago';
	import en from 'javascript-time-ago/locale/en';
	import { getModalStore, getToastStore, type ModalSettings } from '@skeletonlabs/skeleton';
	import {
		PlusCircle,
		ChatBubbleLeftRight,
		Share,
		Trash
	} from '@inqling/svelte-icons/heroicon-24-solid';
	import {
		ChatBubbleBottomCenter,
		AcademicCap,
		Clock
	} from '@inqling/svelte-icons/heroicon-24-outline';
	import { goto } from '$app/navigation';
	import { createNewChat, showToast } from '$misc/shared';
	import { chatStore, isTimeagoInitializedStore } from '$misc/stores';
	import { getCustomContexts } from '$misc/customContexts';

	const modalStore = getModalStore();
	const toastStore = getToastStore();

	let timeAgo: TimeAgo;

	$: sortedChats = Object.entries($chatStore).sort((a, b) => {
		return new Date(b[1].created).getTime() - new Date(a[1].created).getTime();
	});

	onMount(() => {
		if (!$isTimeagoInitializedStore) {
			TimeAgo.addDefaultLocale(en);
			$isTimeagoInitializedStore = true;
		}
		timeAgo = new TimeAgo('en-US');
	});

	async function modalConfirmDelete() {
		const modal: ModalSettings = {
			type: 'confirm',
			title: 'Please confirm',
			body: 'Are you sure you want to delete all chats?',
			response: (result: boolean) => {
				if (result) {
					clearStorage();
				}
			}
		};
		modalStore.trigger(modal);
	}

	async function clearStorage() {
		const docsToDelete: { [key: string]: string } = {};
		let savedChatsAmount = 0;
		for (const [slug, chat] of Object.entries($chatStore)) {
			if (chat.updateToken?.length) {
				docsToDelete[slug] = chat.updateToken;
			}
			savedChatsAmount++;
		}

		// update the local store
		$chatStore = {};

		goto('/');
	}

	let chatTypes = [ 'general doctor', 'therapy', 'diagnose',  'relaxation', 'girlfriend', 'suicide prevention', ];
	let chatTypeDescriptions = {
		'general doctor': 'talk to a generalist doctor who can help you with simple health issues and give advice',
		'therapy': 'talk to a therapist as they help you work your way out of your mental issues',
		'diagnose': 'get an accurate diagnosis of any disease you might have just by inputting your symptoms',
		'girlfriend': 'talk to an online 24/7 e-girl whos more than happy to be your girlfriend. (known to improve mental health!)',
		'suicide prevention': 'theres always something you can do, and talking will help you find alternate methods of coping',
		'relaxation': 'talk to a fun and chill buddy who can help you relax, and soothe your nerves',
	}
	let imageNames = {		
		'general doctor': 'generalDoctor.jpg',
		'therapy': 'therapy.jpg',
		'diagnose': 'diagnose.webp',
		'girlfriend': 'girlfriend.jpg',
		'suicide prevention': 'suicidePrevention.png',
		'relaxation': 'relaxation.jpg',
	}
</script>


<h1 style="padding: 1% 5%">
	Welcome to DigiCare, a cutting-edge medical AI chat app designed to provide remote assistance and support to patients. This innovative application leverages artificial intelligence to offer personalized and reliable health information, fostering a seamless connection between patients and their healthcare providers. Chat with a variety of doctor types to help you with any condition you may have.
</h1>

<div
	class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-5 gap-6 px-4 md:px-8"
>

	{#each chatTypes as chatType}
		<button
			class="card p-4 grid variant-ghost-primary"
			on:click={() => createNewChat({ context: getCustomContexts(chatType), title: `${chatType} chat ${crypto.randomUUID().substr(0, 8)}` })}
		>
		<img src="/{imageNames[chatType]}" style="margin:auto">
			<div class="flex space-x-2 md:space-x-4 items-center self-center justify-self-center">
				<PlusCircle class="w-10 h-10" />
				<span class="text-xl">{chatType}</span>
			</div>
			{chatTypeDescriptions[chatType]}
		</button>
	{/each}

	{#each sortedChats as [slug, chat]}
		<a href={`/${slug}`} class="card p-4 flex flex-col variant-ghost-surface justify-end">
			<div class="flex flex-col">
				<div class="flex items-center space-x-2">
					<div>
						{#if chat.updateToken}
							<!-- Shared -->
							<Share class="w-10 h-10" />
						{:else}
							<!-- Local -->
							<ChatBubbleLeftRight class="w-10 h-10" />
						{/if}
					</div>
					<h2 class="h2 line-clamp-2 text-lg font-bold">{chat.title}</h2>
				</div>
			</div>
			<hr class="my-4" />
			<footer class="flex justify-evenly space-x-2">
				<div class="badge variant-filled-surface flex items-center space-x-1">
					<ChatBubbleBottomCenter class="w-6 h-6" />
					<span>{chatStore.countAllMessages(chat)}</span>
				</div>
				<div class="badge variant-filled-surface flex items-center space-x-1">
					<Clock class="w-6 h-6" />
					{#if timeAgo}
						<span class="self-center">
							{timeAgo.format(new Date(chat.created), 'twitter-minute-now')}
						</span>
					{/if}
				</div>
			</footer>
		</a>
	{/each}

	<!-- Clear button -->
	{#if Object.entries($chatStore)?.length}
		<button class="card p-4 grid variant-ghost-error" on:click={modalConfirmDelete}>
			<div class="flex space-x-2 md:space-x-4 items-center self-center justify-self-center">
				<Trash class="w-10 h-10" />
				<span class="text-xl">Clear storage</span>
			</div>
		</button>
	{/if}
</div>

<style lang="postcss">
	.card {
		min-height: 9rem;
	}
</style>
