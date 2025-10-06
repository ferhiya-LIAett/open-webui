<script>
	import { mobile, showArchivedChats, showSidebar, user } from '$lib/stores';
	import { getContext } from 'svelte';

	const i18n = getContext('i18n');

	import UserMenu from '$lib/components/layout/Sidebar/UserMenu.svelte';
	import Notes from '$lib/components/notes/Notes.svelte';
	import Tooltip from '$lib/components/common/Tooltip.svelte';
	import Sidebar from '$lib/components/icons/Sidebar.svelte';
	import { WEBUI_BASE_URL } from '$lib/constants';
	import { theme } from '$lib/stores';
</script>

<nav
	class="fixed top-0 left-0 right-0 h-[64px] z-50 flex items-center px-4 bg-[#F2F2F2] dark:bg-black text-black dark:text-white"
>
	<div class="flex items-center justify-between w-full">
		<!-- LEFT SECTION: Logo + Sidebar toggle + Page title -->
		<div class="flex items-center gap-4">
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

			<!-- Sidebar toggle (mobile only) -->
			{#if $mobile}
				<div class="{$showSidebar ? 'md:hidden' : ''} flex flex-none items-center">
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

			<!-- Page Title -->
			<a href="/notes" class="text-sm font-medium hover:underline whitespace-nowrap">
				{$i18n.t('Notes')}
			</a>
		</div>

		<!-- RIGHT SECTION: User menu -->
		{#if $user}
			<UserMenu
				className="max-w-[240px]"
				role={$user?.role}
				help={true}
				on:show={(e) => {
					if (e.detail === 'archived-chat') showArchivedChats.set(true);
				}}
			>
				<button
					class="select-none flex rounded-xl p-1.5 hover:bg-gray-100 dark:hover:bg-gray-850 transition"
					aria-label="User Menu"
				>
					<img
						src={$user?.profile_image_url}
						class="size-6 object-cover rounded-full"
						alt="User profile"
						draggable="false"
					/>
				</button>
			</UserMenu>
		{/if}
	</div>
</nav>

<!-- Notes content with top padding -->
<div class="pt-[64px] pb-1 flex-1 max-h-full overflow-y-auto @container">
	<Notes />
</div>
