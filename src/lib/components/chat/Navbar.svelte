<script lang="ts">
	import { getContext } from 'svelte';
	import { toast } from 'svelte-sonner';

	import {
		WEBUI_NAME,
		banners,
		chatId,
		config,
		mobile,
		settings,
		showArchivedChats,
		showControls,
		showSidebar,
		temporaryChatEnabled,
		user
	} from '$lib/stores';

	import { slide } from 'svelte/transition';
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';

	import ShareChatModal from '../chat/ShareChatModal.svelte';
	import ModelSelector from '../chat/ModelSelector.svelte';
	import Tooltip from '../common/Tooltip.svelte';
	import Menu from '$lib/components/layout/Navbar/Menu.svelte';
	import UserMenu from '$lib/components/layout/Sidebar/UserMenu.svelte';
	import AdjustmentsHorizontal from '../icons/AdjustmentsHorizontal.svelte';

	import PencilSquare from '../icons/PencilSquare.svelte';
	import Banner from '../common/Banner.svelte';
	import Sidebar from '../icons/Sidebar.svelte';

	import ChatBubbleDotted from '../icons/ChatBubbleDotted.svelte';
	import ChatBubbleDottedChecked from '../icons/ChatBubbleDottedChecked.svelte';

	import EllipsisHorizontal from '../icons/EllipsisHorizontal.svelte';
	import ChatPlus from '../icons/ChatPlus.svelte';
	import ChatCheck from '../icons/ChatCheck.svelte';
	import Knobs from '../icons/Knobs.svelte';
	import { theme } from '$lib/stores';
	import { WEBUI_BASE_URL } from '$lib/constants';
	const i18n = getContext('i18n');

	export let initNewChat: Function;
	export let shareEnabled: boolean = false;

	export let chat;
	export let history;
	export let selectedModels;
	export let showModelSelector = true;

	export let onSaveTempChat: () => {};
	export let archiveChatHandler: (id: string) => void;
	export let moveChatHandler: (id: string, folderId: string) => void;

	let closedBannerIds = [];

	let showShareChatModal = false;
	let showDownloadChatModal = false;
</script>

<ShareChatModal bind:show={showShareChatModal} chatId={$chatId} />

<button
	id="new-chat-button"
	class="hidden"
	on:click={() => {
		initNewChat();
	}}
	aria-label="New Chat"
/>

<nav class="fixed top-0 left-0 right-0 h-[64px] z-50 flex items-center bg-[#F2F2F2] dark:bg-black">
  <div class="flex items-center w-full h-full px-4">
    <!-- Left: Logo + Links -->
    <div class="flex items-center h-full gap-5">
		<div
		class="flex items-center justify-center w-[100px] h-[50px]"
		>
		<img
			crossorigin="anonymous"
			src={($theme === 'dark' || $theme === 'oled-dark')
			? `${WEBUI_BASE_URL}/static/sweco_text_icon-dark.png`
			: `${WEBUI_BASE_URL}/static/sweco_text_icon.png`}
			alt="Sweco logo"
			class="w-full h-full object-contain"
		/>
		</div>

<button
  id="swecogpt-button"
  class="app-button active-app !text-black dark:!text-white dark:!bg-gray-600/40"
  on:click={() => switchPage('swecogpt')}
>
  SwecoGPT
</button>

<button
  id="swecogpt-plus-button"
  class="app-button !text-black dark:!text-white dark:!bg-gray-600/40"
  on:click={() => switchPage('swecogpt-plus')}
>
  SwecoGPT Plus
</button>

<button
  id="assistant-button"
  class="app-button !text-black dark:!text-white dark:!bg-gray-600/40"
  on:click={() => switchPage('assistant')}
>
  Assistant
</button>
    </div>

    <!-- Right: Controls -->
    <div class="ml-auto flex items-center gap-2 h-full">
      {#if $user?.role === 'user' ? ($user?.permissions?.chat?.temporary ?? true) && !($user?.permissions?.chat?.temporary_enforced ?? false) : true}
        {#if !chat?.id}
          <Tooltip content={$i18n.t(`Temporary Chat`)}>
            <button
              class="flex items-center justify-center px-2 py-2 rounded-xl hover:bg-gray-50 dark:hover:bg-gray-850 transition"
              on:click={async () => {
                await temporaryChatEnabled.set(!$temporaryChatEnabled);
                await goto('/');
              }}
            >
              {#if $temporaryChatEnabled}
                <ChatBubbleDottedChecked class="size-4.5" strokeWidth="1.5" />
              {:else}
                <ChatBubbleDotted class="size-4.5" strokeWidth="1.5" />
              {/if}
            </button>
          </Tooltip>
        {:else if $temporaryChatEnabled}
          <Tooltip content={$i18n.t(`Save Chat`)}>
            <button class="flex items-center justify-center px-2 py-2 rounded-xl hover:bg-gray-50 dark:hover:bg-gray-850 transition" on:click={onSaveTempChat}>
              <ChatCheck class="size-4.5" strokeWidth="1.5" />
            </button>
          </Tooltip>
        {/if}
      {/if}

      {#if shareEnabled && chat && (chat.id || $temporaryChatEnabled)}
        <Menu {chat} {shareEnabled} shareHandler={() => (showShareChatModal = !showShareChatModal)} archiveChatHandler={() => archiveChatHandler(chat.id)} {moveChatHandler}>
          <button class="flex items-center justify-center px-2 py-2 rounded-xl hover:bg-gray-50 dark:hover:bg-gray-850 transition">
            <EllipsisHorizontal class="size-5" strokeWidth="1.5" />
          </button>
        </Menu>
      {/if}

      {#if $user?.role === 'admin' || ($user?.permissions.chat?.controls ?? true)}
        <Tooltip content={$i18n.t('Controls')}>
          <button class="flex items-center justify-center px-2 py-2 rounded-xl hover:bg-gray-50 dark:hover:bg-gray-850 transition" on:click={() => showControls.set(!$showControls)}>
            <Knobs class="size-5" strokeWidth="1" />
          </button>
        </Tooltip>
      {/if}

      {#if $user}
        <UserMenu class="max-w-[240px]" role={$user?.role} help={true}>
          <div class="select-none flex items-center rounded-xl p-1.5 hover:bg-gray-50 dark:hover:bg-gray-850 transition">
            <img src={$user?.profile_image_url} class="size-6 object-cover rounded-full" alt="User profile" draggable="false" />
          </div>
        </UserMenu>
      {/if}
    </div>
  </div>
</nav>

<!-- Sidebar toggle buttons and selectmodel -->
 <!-- Sidebar toggle buttons and selectmodel -->
<div
  style="font-weight: bold; z-index: 50; padding-top: 4rem;"
  class="transition-all duration-300 ease-in-out"
  
  class:p-3={!$showSidebar} 
  class:p-[3%]={$showSidebar}
  class:pt-[4rem]={$showSidebar}
  class:pt-6={!$showSidebar}
>
  <div
    class="-translate-x-0.5 mr-1 mt-1 self-start flex flex-none items-center text-gray-600 dark:text-gray-400 z-50 relative gap-4 transition-all duration-300"
  >
    <Tooltip content={$showSidebar ? $i18n.t('Close Sidebar') : $i18n.t('Open Sidebar')}>
      <button
        id="toggle-navbar-button"
        on:click={() => showSidebar.set(!$showSidebar)}
        class="transition duration-300 p-0"
        aria-label={$showSidebar ? $i18n.t('Close Sidebar') : $i18n.t('Open Sidebar')}
      >
        <i
          class="fad fa-chevron-double-right navbar-rotate transition duration-300"
          class:rotate-180={$showSidebar}
          style="
            --fa-primary-color: {$theme === 'dark' || $theme === 'oled-dark' ? 'white' : 'black'};
            --fa-secondary-color: gray;
            font-size: 16px;
            width: 14px;
            height: 16px;
            line-height: 16px;
          "
        ></i>
      </button>
    </Tooltip>

    <!-- Model Selector -->
    {#if showModelSelector}
      <ModelSelector bind:selectedModels showSetDefault={!shareEnabled} />
    {/if}
  </div>
</div>
