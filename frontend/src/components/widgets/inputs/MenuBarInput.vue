<template>
	<div class="menu-bar-input" data-menu-bar-input>
		<div class="entry-container">
			<button @click="() => visitWebsite('https://graphite.rs')" class="entry">
				<IconLabel :icon="'GraphiteLogo'" />
			</button>
		</div>
		<div class="entry-container" v-for="(entry, index) in entries" :key="index">
			<div
				@click="(e) => onClick(entry, e.target)"
				tabindex="0"
				@blur="(e: FocusEvent) => blur(e,entry)"
				@keydown="entry.ref?.keydown"
				class="entry"
				:class="{ open: entry.ref?.isOpen }"
				data-hover-menu-spawner
			>
				<IconLabel v-if="entry.icon" :icon="entry.icon" />
				<span v-if="entry.label">{{ entry.label }}</span>
			</div>
			<MenuList
				:open="entry.ref?.open || false"
				:entries="entry.children || []"
				:direction="'Bottom'"
				:minWidth="240"
				:drawIcon="true"
				:defaultAction="() => editor.instance.request_coming_soon_dialog()"
				:ref="(ref: typeof MenuList) => ref && (entry.ref = ref)"
			/>
		</div>
	</div>
</template>

<style lang="scss">
.menu-bar-input {
	display: flex;

	.entry-container {
		display: flex;
		position: relative;

		.entry {
			display: flex;
			align-items: center;
			white-space: nowrap;
			padding: 0 8px;
			background: none;
			border: 0;
			margin: 0;

			svg {
				fill: var(--color-e-nearwhite);
			}

			&:hover,
			&.open {
				background: var(--color-6-lowergray);

				svg {
					fill: var(--color-f-white);
				}

				span {
					color: var(--color-f-white);
				}
			}
		}
	}
}
</style>

<script lang="ts">
import { defineComponent } from "vue";

import { Editor } from "@/wasm-communication/editor";

import MenuList, { MenuListEntry, MenuListEntries } from "@/components/floating-menus/MenuList.vue";
import IconLabel from "@/components/widgets/labels/IconLabel.vue";

function makeEntries(editor: Editor): MenuListEntries {
	return [
		{
			label: "File",
			ref: undefined,
			children: [
				[
					{ label: "New…", icon: "File", shortcut: ["KeyControl", "KeyN"], shortcutRequiresLock: true, action: (): void => editor.instance.request_new_document_dialog() },
					{ label: "Open…", shortcut: ["KeyControl", "KeyO"], action: (): void => editor.instance.open_document() },
					{
						label: "Open Recent",
						shortcut: ["KeyControl", "KeyShift", "KeyO"],
						action: (): void => undefined,
						children: [
							[{ label: "Reopen Last Closed", shortcut: ["KeyControl", "KeyShift", "KeyT"], shortcutRequiresLock: true }, { label: "Clear Recently Opened" }],
							[
								{ label: "Some Recent File.gdd" },
								{ label: "Another Recent File.gdd" },
								{ label: "An Older File.gdd" },
								{ label: "Some Other Older File.gdd" },
								{ label: "Yet Another Older File.gdd" },
							],
						],
					},
				],
				[
					{ label: "Close", shortcut: ["KeyControl", "KeyW"], shortcutRequiresLock: true, action: async (): Promise<void> => editor.instance.close_active_document_with_confirmation() },
					{ label: "Close All", shortcut: ["KeyControl", "KeyAlt", "KeyW"], action: async (): Promise<void> => editor.instance.close_all_documents_with_confirmation() },
				],
				[
					{ label: "Save", shortcut: ["KeyControl", "KeyS"], action: async (): Promise<void> => editor.instance.save_document() },
					{ label: "Save As…", shortcut: ["KeyControl", "KeyShift", "KeyS"], action: async (): Promise<void> => editor.instance.save_document() },
					{ label: "Save All", shortcut: ["KeyControl", "KeyAlt", "KeyS"] },
					{ label: "Auto-Save", checkbox: true, checked: true },
				],
				[
					{ label: "Import…", shortcut: ["KeyControl", "KeyI"] },
					{ label: "Export…", shortcut: ["KeyControl", "KeyE"], action: async (): Promise<void> => editor.instance.export_document() },
				],
				[{ label: "Quit", shortcut: ["KeyControl", "KeyQ"] }],
			],
		},
		{
			label: "Edit",
			ref: undefined,
			children: [
				[
					{ label: "Undo", shortcut: ["KeyControl", "KeyZ"], action: async (): Promise<void> => editor.instance.undo() },
					{ label: "Redo", shortcut: ["KeyControl", "KeyShift", "KeyZ"], action: async (): Promise<void> => editor.instance.redo() },
				],
				[
					{ label: "Cut", shortcut: ["KeyControl", "KeyX"], action: async (): Promise<void> => editor.instance.cut() },
					{ label: "Copy", icon: "Copy", shortcut: ["KeyControl", "KeyC"], action: async (): Promise<void> => editor.instance.copy() },
					// TODO: Fix this
					// { label: "Paste", icon: "Paste", shortcut: ["KeyControl", "KeyV"], action: async (): Promise<void> => editor.instance.paste() },
				],
			],
		},
		{
			label: "Layer",
			ref: undefined,
			children: [
				[
					{ label: "Select All", shortcut: ["KeyControl", "KeyA"], action: async (): Promise<void> => editor.instance.select_all_layers() },
					{ label: "Deselect All", shortcut: ["KeyControl", "KeyAlt", "KeyA"], action: async (): Promise<void> => editor.instance.deselect_all_layers() },
					{
						label: "Order",
						action: (): void => undefined,
						children: [
							[
								{
									label: "Raise To Front",
									shortcut: ["KeyControl", "KeyShift", "KeyLeftBracket"],
									action: async (): Promise<void> => editor.instance.reorder_selected_layers(editor.instance.i32_max()),
								},
								{ label: "Raise", shortcut: ["KeyControl", "KeyRightBracket"], action: async (): Promise<void> => editor.instance.reorder_selected_layers(1) },
								{ label: "Lower", shortcut: ["KeyControl", "KeyLeftBracket"], action: async (): Promise<void> => editor.instance.reorder_selected_layers(-1) },
								{
									label: "Lower to Back",
									shortcut: ["KeyControl", "KeyShift", "KeyRightBracket"],
									action: async (): Promise<void> => editor.instance.reorder_selected_layers(editor.instance.i32_min()),
								},
							],
						],
					},
				],
			],
		},
		{
			label: "Document",
			ref: undefined,
			children: [[{ label: "Menu entries coming soon" }]],
		},
		{
			label: "View",
			ref: undefined,
			children: [
				[
					{
						label: "Show/Hide Node Graph (In Development)",
						action: async (): Promise<void> => editor.instance.toggle_node_graph_visibility(),
					},
				],
			],
		},
		{
			label: "Help",
			ref: undefined,
			children: [
				[
					{
						label: "About Graphite",
						action: async (): Promise<void> => editor.instance.request_about_graphite_dialog(),
					},
				],
				[
					{ label: "Report a Bug", action: (): unknown => window.open("https://github.com/GraphiteEditor/Graphite/issues/new", "_blank") },
					{ label: "Visit on GitHub", action: (): unknown => window.open("https://github.com/GraphiteEditor/Graphite", "_blank") },
				],
				[
					{
						label: "Debug: Set Log Level",
						action: (): void => undefined,
						children: [
							[
								{ label: "Log Level Info", action: async (): Promise<void> => editor.instance.log_level_info(), shortcut: ["Key1"] },
								{ label: "Log Level Debug", action: async (): Promise<void> => editor.instance.log_level_debug(), shortcut: ["Key2"] },
								{ label: "Log Level Trace", action: async (): Promise<void> => editor.instance.log_level_trace(), shortcut: ["Key3"] },
							],
						],
					},
					{ label: "Debug: Panic (DANGER)", action: async (): Promise<void> => editor.instance.intentional_panic() },
				],
			],
		},
	];
}

export default defineComponent({
	inject: ["editor"],
	methods: {
		onClick(menuEntry: MenuListEntry, target: EventTarget | null) {
			// Focus the target so that keyboard inputs are sent to the dropdown
			(target as HTMLElement)?.focus();

			if (menuEntry.ref) menuEntry.ref.isOpen = true;
			else throw new Error("The menu bar floating menu has no associated ref");
		},
		blur(e: FocusEvent, menuEntry: MenuListEntry) {
			if ((e.target as HTMLElement).closest("[data-menu-bar-input]") !== this.$el && menuEntry.ref) menuEntry.ref.isOpen = false;
		},
		// TODO: Move to backend
		visitWebsite(url: string) {
			// This method is required because `window` isn't accessible from the Vue component HTML
			window.open(url, "_blank");
		},
	},
	data() {
		return {
			entries: makeEntries(this.editor),
			open: false,
		};
	},
	components: {
		IconLabel,
		MenuList,
	},
});
</script>
