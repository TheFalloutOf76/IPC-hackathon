<script lang="ts">
	import 'highlightjs-copy/dist/highlightjs-copy.min.css';
	import '../app.postcss';
	import { inject } from '@vercel/analytics';

	import { dev } from '$app/environment';
	import {
		type ModalComponent,
		AppShell,
		Modal,
		storeHighlightJs,
		Toast,
		storePopup,
		setInitialClassState
	} from '@skeletonlabs/skeleton';
	import hljs from 'highlight.js';
	import CopyButtonPlugin from 'highlightjs-copy';
	import { computePosition, autoUpdate, flip, shift, offset, arrow } from '@floating-ui/dom';
	import 'highlight.js/styles/night-owl.css';

	import Header from '$lib/Header.svelte';
	import SettingsModal from '$lib/Modals/SettingsModal.svelte';
	import ContextModal from '$lib/Modals/ContextModal.svelte';
	import ShareModal from '$lib/Modals/ShareModal.svelte';
	import CostModal from '$lib/Modals/CostModal.svelte';
	import SuggestTitleModal from '$lib/Modals/SuggestTitleModal.svelte';
	import { initializeStores } from '@skeletonlabs/skeleton';

	inject({ mode: dev ? 'development' : 'production' });

	hljs.addPlugin(new CopyButtonPlugin());

	initializeStores();
	setupSkeleton();

	// see https://www.skeleton.dev/utilities/modals
	const modalComponentRegistry: Record<string, ModalComponent> = {
		SettingsModal: { ref: SettingsModal },
		ContextModal: { ref: ContextModal },
		ShareModal: { ref: ShareModal },
		CostModal: { ref: CostModal },
		SuggestTitleModal: { ref: SuggestTitleModal }
	};

	function setupSkeleton() {
		setInitialClassState();
		storeHighlightJs.set(hljs);
		storePopup.set({ computePosition, autoUpdate, flip, shift, offset, arrow });
	}
</script>

<svelte:head>
	<title>Girlfriend Therapy</title>
</svelte:head>

<AppShell
	regionPage="relative lg:px-12"
	slotHeader="py-2 md:py-6 px-4 lg:px-12"
	slotFooter="py-2 md:py-6 px-4 lg:px-12"
>
	<svelte:fragment slot="header">
		<Header />
	</svelte:fragment>

	<slot />
</AppShell>

<!-- Skeleton Singletons: -->
<Toast />

<Modal components={modalComponentRegistry} />
