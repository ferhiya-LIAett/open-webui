<script lang="ts">
	import { models, settings, user } from '$lib/stores';
	import { getContext } from 'svelte';
	import Selector from './ModelSelector/Selector.svelte';
	import Tooltip from '../common/Tooltip.svelte';
	import { toast } from 'svelte-sonner';
	import { updateUserSettings } from '$lib/apis/users';

	const i18n = getContext('i18n');

	export let selectedModels: string[] = ['']; // start empty
	export let disabled = false;
	export let showSetDefault = true;

	// Save default model to user settings
	const saveDefaultModel = async () => {
		const hasEmpty = selectedModels.some((m) => m === '');
		if (hasEmpty) {
			toast.error('Välj en modell innan du sparar.');
			return;
		}

		settings.set({ ...$settings, models: selectedModels });
		await updateUserSettings(localStorage.token, { ui: $settings });
		toast.success('Standardmodell uppdaterad');
	};

	// Pin or unpin a model
	const pinModelHandler = async (modelId) => {
		let pinned = $settings?.pinnedModels ?? [];
		pinned = pinned.includes(modelId)
			? pinned.filter((id) => id !== modelId)
			: [...new Set([...pinned, modelId])];

		settings.set({ ...$settings, pinnedModels: pinned });
		await updateUserSettings(localStorage.token, { ui: $settings });
	};

	// Ensure only valid models are selected; invalid ones stay empty
	$: selectedModels = selectedModels.map((model) =>
		model && $models.find((m) => m.id === model) ? model : ''
	);
</script>

<div class="flex flex-col w-full items-start">
	{#each selectedModels as selectedModel, idx}
		<div class="flex w-full max-w-fit">
			<div class="overflow-hidden w-full">
				<Selector
					id={`${idx}`}
					placeholder="Välj modell"
					items={$models.map((m) => ({ value: m.id, label: m.name, model: m }))}
					{pinModelHandler}
					bind:value={selectedModels[idx]}
				/>
			</div>

			{#if $user?.role === 'admin' || ($user?.permissions?.chat?.multiple_models ?? true)}
				{#if idx === 0}
					<div class="self-center mx-1">
						<Tooltip content="Lägg till modell">
							<button {disabled} on:click={() => selectedModels = [...selectedModels, '']}>+</button>
						</Tooltip>
					</div>
				{:else}
					<div class="self-center mx-1">
						<Tooltip content="Ta bort modell">
							<button {disabled} on:click={() => { selectedModels.splice(idx, 1); selectedModels = selectedModels; }}>−</button>
						</Tooltip>
					</div>
				{/if}
			{/if}
		</div>
	{/each}
</div>

{#if showSetDefault}
	<div class="relative text-left mt-[1px] ml-1 text-[0.7rem] text-gray-600 dark:text-gray-400 font-primary">
		<button on:click={saveDefaultModel}>Sätt som standard</button>
	</div>
{/if}
