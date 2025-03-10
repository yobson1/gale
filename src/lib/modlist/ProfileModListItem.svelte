<script lang="ts" context="module">
	import { writable } from 'svelte/store';

	export let activeContextMenu = writable<string | null>(null);
</script>

<script lang="ts">
	import type { Mod, ModContextItem } from '../models';
	import Icon from '@iconify/svelte';
	import { iconSrc, isOutdated } from '$lib/util';
	import { Switch, ContextMenu } from 'bits-ui';
	import { createEventDispatcher } from 'svelte';
	import ModContextMenuItems from './ModContextMenuItems.svelte';
	import { dropTransition } from '$lib/transitions';

	export let mod: Mod;
	export let index: number;
	export let isSelected: boolean;
	export let contextItems: ModContextItem[];

	export let reorderable: boolean;

	const dispatch = createEventDispatcher<{
		toggle: boolean;
	}>();

	let contextMenuOpen: boolean;

	$: descriptionClasses =
		mod.enabled === false
			? 'text-slate-500 line-through'
			: isSelected
				? 'text-slate-300'
				: 'text-slate-400 group-hover:text-slate-300';

	$: if ($activeContextMenu !== null && $activeContextMenu !== mod.uuid) {
		contextMenuOpen = false;
	}
</script>

<ContextMenu.Root
	bind:open={contextMenuOpen}
	onOpenChange={(open) => {
		if (open) {
			$activeContextMenu = mod.uuid;
		} else {
			$activeContextMenu = null;
		}
	}}
>
	<ContextMenu.Trigger class="contents">
		<button
			class="group flex w-full items-center rounded-lg border p-2 {isSelected
				? 'border-slate-500 bg-slate-700'
				: 'border-transparent hover:bg-slate-700'}"
			data-uuid={mod.uuid}
			data-index={index}
			draggable={reorderable}
			on:click
			on:dragstart
			on:dragover
			on:dragend
		>
			<img src={iconSrc(mod)} alt={mod.name} class="size-12 rounded-sm" />
			<div class="shrink grow overflow-hidden pr-2 pl-3 text-left">
				<div class="flex items-center gap-1 overflow-hidden">
					<div
						class="shrink truncate font-medium {mod.enabled === false
							? 'text-slate-300 line-through'
							: 'text-white'}"
					>
						{mod.name.replace(/_/g, ' ')}
					</div>
					<div class="px-1 {descriptionClasses}">
						{mod.version ?? '?.?.?'}
					</div>
					{#if mod.isPinned}
						<Icon class="shrink-0 text-slate-400" icon="mdi:pin" />
					{/if}
					{#if mod.isDeprecated}
						<Icon class="shrink-0 text-red-500" icon="mdi:error" />
					{/if}
					{#if isOutdated(mod)}
						<Icon class="text-accent-500 shrink-0" icon="mdi:arrow-up-circle" />
					{/if}
				</div>

				{#if mod.description !== null}
					<div class="truncate {descriptionClasses}">
						{mod.description}
					</div>
				{/if}
			</div>

			{#if reorderable}
				<Icon
					icon="material-symbols:drag-indicator"
					class="mr-4 shrink-0 cursor-move text-2xl text-slate-400"
				/>
			{/if}

			<!-- make sure click events don't propagate and cause the mod to be selected -->
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<!-- svelte-ignore a11y-no-static-element-interactions -->
			<div class="contents" on:click={(evt) => evt.stopPropagation()}>
				<Switch.Root
					checked={mod.enabled ?? true}
					onCheckedChange={(newState) => dispatch('toggle', newState)}
					class="group data-[state=checked]:bg-accent-700 data-[state=checked]:hover:bg-accent-600 mr-1 flex h-6 w-12 shrink-0 rounded-full bg-slate-600 px-1 py-1 hover:bg-slate-500"
				>
					<Switch.Thumb
						class="data-[state=checked]:bg-accent-200 pointer-events-none h-full w-4 rounded-full bg-slate-300 transition-transform duration-75 ease-out data-[state=checked]:translate-x-6"
					/>
				</Switch.Root>
			</div>
		</button>
	</ContextMenu.Trigger>
	<ContextMenu.Content
		class="flex flex-col gap-0.5 rounded-lg border border-slate-600 bg-slate-800 p-1 shadow-lg"
		{...dropTransition}
	>
		<ModContextMenuItems {mod} {contextItems} type="context" />
	</ContextMenu.Content>
</ContextMenu.Root>
