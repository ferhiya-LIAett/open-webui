<script lang="ts">
	import { onMount, getContext } from 'svelte';
	import {
		WEBUI_NAME,
		showSidebar,
		functions,
		user,
		mobile,
		models,
		prompts,
		knowledge,
		tools
	} from '$lib/stores';
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';
	import Tooltip from '$lib/components/common/Tooltip.svelte';
	import Sidebar from '$lib/components/icons/Sidebar.svelte';
	import { theme } from '$lib/stores';
	import { WEBUI_BASE_URL } from '$lib/constants'

	const i18n = getContext('i18n');

	let loaded = false;

	onMount(async () => {
		if ($user?.role !== 'admin') {
			if ($page.url.pathname.includes('/models') && !$user?.permissions?.workspace?.models) {
				goto('/');
			} else if (
				$page.url.pathname.includes('/knowledge') &&
				!$user?.permissions?.workspace?.knowledge
			) {
				goto('/');
			} else if (
				$page.url.pathname.includes('/prompts') &&
				!$user?.permissions?.workspace?.prompts
			) {
				goto('/');
			} else if ($page.url.pathname.includes('/tools') && !$user?.permissions?.workspace?.tools) {
				goto('/');
			}
		}

		loaded = true;
	});
</script>

<svelte:head>
	<title>
		{$i18n.t('Workspace')} • {$WEBUI_NAME}
	</title>
</svelte:head>

{#if loaded}
<div
	class="relative flex flex-col w-full h-screen max-h-[100dvh] transition-width duration-200 ease-in-out {$showSidebar
		? 'md:max-w-[calc(100%-260px)]'
		: ''} max-w-full"
>
	<!-- Navbar -->
	<nav
		class="fixed top-0 left-0 right-0 h-[64px] z-50 flex items-center px-4 bg-[#F2F2F2] dark:bg-black text-black dark:text-white"
	>
		<div class="flex items-center justify-between w-full">
			<!-- LEFT SECTION -->
			 <!-- Sweco Logo -->
			<div class="flex items-center justify-center w-[100px] h-[50px]">
				<img
					crossorigin="anonymous"
					src={($theme === 'dark' || $theme === 'oled-dark')
						? `${WEBUI_BASE_URL}/static/sweco_text_icon-dark.png`
						: `${WEBUI_BASE_URL}/static/sweco_text_icon.png`}
					alt="Sweco logo"
					class="w-full h-full object-contain"
				/>
			</div>
			<div class="flex items-center gap-4">
				<!-- Sidebar toggle (mobile only) -->
				{#if $mobile}
					<div class="{$showSidebar ? 'md:hidden' : ''} self-center flex flex-none items-center">
						<Tooltip
							content={$showSidebar ? $i18n.t('Close Sidebar') : $i18n.t('Open Sidebar')}
							interactive={true}
						>
							<button
								id="sidebar-toggle-button"
								class="cursor-pointer flex rounded-lg hover:bg-gray-100 dark:hover:bg-gray-850 transition"
								on:click={() => {
									showSidebar.set(!$showSidebar);
								}}
							>
								<div class="self-center p-1.5">
									<Sidebar />
								</div>
							</button>
						</Tooltip>
					</div>
				{/if}

				<!-- Workspace Navigation Links -->
				<div class="flex gap-2 items-center text-sm font-medium">
					{#if $user?.role === 'admin' || $user?.permissions?.workspace?.models}
						<a
							class="px-2 py-1.5 rounded-lg transition-all duration-200 {$page.url.pathname.includes('/workspace/models')
								? 'bg-gray-300 dark:bg-gray-800 text-black dark:text-white'
								: 'text-gray-500 dark:text-gray-400 hover:text-black dark:hover:text-white hover:bg-gray-100 dark:hover:bg-gray-850'}"
							href="/workspace/models"
						>
							{$i18n.t('Models')}
						</a>
					{/if}

					{#if $user?.role === 'admin' || $user?.permissions?.workspace?.knowledge}
						<a
							class="px-2 py-1.5 rounded-lg transition-all duration-200 {$page.url.pathname.includes('/workspace/knowledge')
								? 'bg-gray-300 dark:bg-gray-800 text-black dark:text-white'
								: 'text-gray-500 dark:text-gray-400 hover:text-black dark:hover:text-white hover:bg-gray-100 dark:hover:bg-gray-850'}"
							href="/workspace/knowledge"
						>
							{$i18n.t('Knowledge')}
						</a>
					{/if}

					{#if $user?.role === 'admin' || $user?.permissions?.workspace?.prompts}
						<a
							class="px-2 py-1.5 rounded-lg transition-all duration-200 {$page.url.pathname.includes('/workspace/prompts')
								? 'bg-gray-300 dark:bg-gray-800 text-black dark:text-white'
								: 'text-gray-500 dark:text-gray-400 hover:text-black dark:hover:text-white hover:bg-gray-100 dark:hover:bg-gray-850'}"
							href="/workspace/prompts"
						>
							{$i18n.t('Prompts')}
						</a>
					{/if}

					{#if $user?.role === 'admin' || $user?.permissions?.workspace?.tools}
						<a
							class="px-2 py-1.5 rounded-lg transition-all duration-200 {$page.url.pathname.includes('/workspace/tools')
								? 'bg-gray-300 dark:bg-gray-800 text-black dark:text-white'
								: 'text-gray-500 dark:text-gray-400 hover:text-black dark:hover:text-white hover:bg-gray-100 dark:hover:bg-gray-850'}"
							href="/workspace/tools"
						>
							{$i18n.t('Tools')}
						</a>
					{/if}
				</div>
			</div>
		</div>
	</nav>

	<!-- MAIN CONTENT -->
	<div class="pt-[64px] pb-1 px-[18px] flex-1 max-h-full overflow-y-auto" id="workspace-container">
		<slot />
	</div>
</div>

{/if}
