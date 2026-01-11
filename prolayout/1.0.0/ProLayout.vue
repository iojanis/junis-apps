<!-- @app {"title":"ProLayout","icon":"icons/apps/org.gnome.Extensions.svg","width":1200,"height":720,"singleWindow":false, "inset":true} -->
<template>
    <div class="h-full flex flex-col">
        <Menubar @mousedown.stop class="border-b shrink-0">
            <MenubarMenu class="menubar-menu pointer-events-auto">
                <MenubarTrigger class="menubar-trigger">File</MenubarTrigger>
                <MenubarContent>
                    <MenubarItem
                        @click="createNewItem"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <FileText class="mr-2 h-4 w-4" />
                            New Item
                        </div>
                        <MenubarShortcut>Ctrl+N</MenubarShortcut>
                    </MenubarItem>
                    <MenubarSeparator />
                    <MenubarItem
                        @click="openFolder"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <FolderOpen class="mr-2 h-4 w-4" />
                            Open Folder
                        </div>
                        <MenubarShortcut>Ctrl+O</MenubarShortcut>
                    </MenubarItem>
                    <MenubarSeparator />
                    <MenubarItem
                        @click="saveItem"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <Save class="mr-2 h-4 w-4" />
                            Save
                        </div>
                        <MenubarShortcut>Ctrl+S</MenubarShortcut>
                    </MenubarItem>
                </MenubarContent>
            </MenubarMenu>

            <MenubarMenu class="menubar-menu">
                <MenubarTrigger class="menubar-trigger">Edit</MenubarTrigger>
                <MenubarContent>
                    <MenubarItem
                        @click="undo"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <Undo class="mr-2 h-4 w-4" />
                            Undo
                        </div>
                        <MenubarShortcut>Ctrl+Z</MenubarShortcut>
                    </MenubarItem>
                    <MenubarItem
                        @click="redo"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <Redo class="mr-2 h-4 w-4" />
                            Redo
                        </div>
                        <MenubarShortcut>Ctrl+Y</MenubarShortcut>
                    </MenubarItem>
                    <MenubarSeparator />
                    <MenubarItem
                        @click="selectAll"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <MousePointer class="mr-2 h-4 w-4" />
                            Select All
                        </div>
                        <MenubarShortcut>Ctrl+A</MenubarShortcut>
                    </MenubarItem>
                </MenubarContent>
            </MenubarMenu>

            <MenubarMenu class="menubar-menu">
                <MenubarTrigger class="menubar-trigger">View</MenubarTrigger>
                <MenubarContent>
                    <MenubarItem
                        @click="toggleLeftSidebar"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <Check
                                v-if="showLeftSidebar"
                                class="mr-2 h-4 w-4"
                            />
                            <span class="ml-6" v-else />
                            <PanelLeft class="mr-2 h-4 w-4" />
                            {{ showLeftSidebar ? "Hide" : "Show" }} Left Panel
                        </div>
                        <MenubarShortcut>Ctrl+B</MenubarShortcut>
                    </MenubarItem>
                    <MenubarItem
                        @click="toggleRightSidebar"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <Check
                                v-if="showRightSidebar"
                                class="mr-2 h-4 w-4"
                            />
                            <span class="ml-6" v-else />
                            <PanelRight class="mr-2 h-4 w-4" />
                            {{ showRightSidebar ? "Hide" : "Show" }} Right Panel
                        </div>
                        <MenubarShortcut>Ctrl+Shift+B</MenubarShortcut>
                    </MenubarItem>
                    <MenubarSeparator />
                    <MenubarItem
                        @click="zoomIn"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <ZoomIn class="mr-2 h-4 w-4" />
                            Zoom In
                        </div>
                        <MenubarShortcut>Ctrl+=</MenubarShortcut>
                    </MenubarItem>
                    <MenubarItem
                        @click="zoomOut"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <ZoomOut class="mr-2 h-4 w-4" />
                            Zoom Out
                        </div>
                        <MenubarShortcut>Ctrl+-</MenubarShortcut>
                    </MenubarItem>
                    <MenubarItem
                        @click="resetZoom"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <RotateCcw class="mr-2 h-4 w-4" />
                            Reset Zoom
                        </div>
                        <MenubarShortcut>Ctrl+0</MenubarShortcut>
                    </MenubarItem>
                </MenubarContent>
            </MenubarMenu>

            <MenubarMenu class="menubar-menu">
                <MenubarTrigger class="menubar-trigger">Help</MenubarTrigger>
                <MenubarContent>
                    <MenubarItem
                        @click="showShortcutsDialog"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <Keyboard class="mr-2 h-4 w-4" />
                            Keyboard Shortcuts
                        </div>
                        <MenubarShortcut>Ctrl+K Ctrl+S</MenubarShortcut>
                    </MenubarItem>
                </MenubarContent>
            </MenubarMenu>
        </Menubar>

        <div class="flex-grow pointer-events-auto flex overflow-hidden">
            <ResizablePanelGroup
                direction="horizontal"
                @layout="handleMainLayoutChange"
            >
                <!-- Left Sidebar Panel -->
                <ResizablePanel
                    ref="leftSidebarPanel"
                    :default-size="showLeftSidebar ? 20 : 0"
                    :min-size="15"
                    :collapsed-size="0"
                    collapsible
                >
                    <div class="h-full bg-white/40 dark:bg-zinc-900/40 w-full">
                        <div class="sidebar-header p-3 border-b">
                            <h3 class="text-sm font-semibold">Explorer</h3>
                        </div>
                        <ScrollArea class="h-full">
                            <div @mousedown.stop class="p-2">
                                <div
                                    class="flex items-center justify-between mb-2"
                                >
                                    <div class="flex items-center gap-1">
                                        <Button
                                            variant="ghost"
                                            size="sm"
                                            @click="createNewItem"
                                            v-tippy="{
                                                content: 'New Item (Ctrl+N)',
                                                theme: 'translucent',
                                            }"
                                        >
                                            <Plus class="h-4 w-4" />
                                        </Button>
                                        <Button
                                            variant="ghost"
                                            size="sm"
                                            @click="refreshTree"
                                            v-tippy="{
                                                content: 'Refresh Tree',
                                                theme: 'translucent',
                                            }"
                                        >
                                            <RefreshCw class="h-4 w-4" />
                                        </Button>
                                    </div>
                                </div>

                                <!-- Simple Tree View -->
                                <TreeRoot
                                    v-slot="{ flattenItems }"
                                    class="list-none select-none w-full text-blackA11 rounded-lg text-sm font-medium"
                                    :items="treeItems"
                                    :get-key="(item) => item.id"
                                    :default-expanded="expandedItems"
                                    :model-value="
                                        selectedItemId ? [selectedItemId] : []
                                    "
                                    @update:model-value="updateSelection"
                                    multiple
                                >
                                    <TreeItem
                                        v-for="item in flattenItems"
                                        v-slot="{ isExpanded, handleToggle }"
                                        :key="item.value.id"
                                        :style="{
                                            'padding-left': `${item.level * 0.5}rem`,
                                        }"
                                        v-bind="item.bind"
                                        :class="[
                                            'tree-item flex items-center py-1 px-2 my-0.5 rounded outline-none cursor-pointer transition-colors',
                                            selectedItemId === item.value.id
                                                ? 'bg-blue-500/20 dark:bg-blue-400/30 text-blue-700 dark:text-blue-300'
                                                : 'hover:bg-zinc-100 dark:hover:bg-zinc-800',
                                            'focus:ring-2 focus:ring-white/50',
                                        ]"
                                        @contextmenu.prevent="
                                            openContextMenu($event, item)
                                        "
                                        @click="handleItemClick(item)"
                                    >
                                        <ContextMenu
                                            :open="
                                                contextMenuOpen &&
                                                selectedTreeItem?.value?.id ===
                                                    item.value.id
                                            "
                                            @update:open="
                                                contextMenuOpen = $event
                                            "
                                        >
                                            <ContextMenuTrigger class="w-full">
                                                <div
                                                    class="flex items-center gap-2 w-full"
                                                >
                                                    <span
                                                        class="flex-shrink-0"
                                                        v-if="
                                                            item.value.type ===
                                                                'folder' &&
                                                            item.value
                                                                .children &&
                                                            item.value.children
                                                                .length > 0
                                                        "
                                                    >
                                                        <ChevronRight
                                                            :class="[
                                                                'h-3 w-3 transition-transform duration-200',
                                                                isExpanded
                                                                    ? 'rotate-90'
                                                                    : '',
                                                            ]"
                                                            @click.stop="
                                                                handleToggle()
                                                            "
                                                        />
                                                    </span>
                                                    <span
                                                        v-else
                                                        class="w-3"
                                                    ></span>

                                                    <span class="flex-shrink-0">
                                                        <FileText
                                                            v-if="
                                                                item.value
                                                                    .type ===
                                                                'item'
                                                            "
                                                            class="h-4 w-4 text-blue-600 dark:text-blue-400"
                                                        />
                                                        <FolderOpen
                                                            v-if="
                                                                item.value
                                                                    .type ===
                                                                    'folder' &&
                                                                isExpanded
                                                            "
                                                            class="h-4 w-4 text-amber-600 dark:text-amber-400"
                                                        />
                                                        <Folder
                                                            v-else-if="
                                                                item.value
                                                                    .type ===
                                                                'folder'
                                                            "
                                                            class="h-4 w-4 text-amber-600 dark:text-amber-400"
                                                        />
                                                    </span>

                                                    <span
                                                        class="flex-1 truncate"
                                                    >
                                                        {{ item.value.name }}
                                                    </span>
                                                </div>
                                            </ContextMenuTrigger>
                                            <ContextMenuContent class="w-48">
                                                <ContextMenuItem
                                                    @click="createNewItem"
                                                >
                                                    <Plus
                                                        class="mr-3 h-4 w-4"
                                                    />
                                                    New Item
                                                </ContextMenuItem>
                                                <ContextMenuItem
                                                    @click="createNewFolder"
                                                    v-if="
                                                        item.value.type ===
                                                        'folder'
                                                    "
                                                >
                                                    <FolderPlus
                                                        class="mr-3 h-4 w-4"
                                                    />
                                                    New Folder
                                                </ContextMenuItem>
                                                <ContextMenuSeparator />
                                                <ContextMenuItem
                                                    @click="startRename"
                                                >
                                                    <Edit
                                                        class="mr-3 h-4 w-4"
                                                    />
                                                    Rename
                                                </ContextMenuItem>
                                                <ContextMenuSeparator />
                                                <ContextMenuItem
                                                    class="text-red-500 focus:text-red-500"
                                                    @click="deleteSelected"
                                                >
                                                    <Trash2
                                                        class="mr-3 h-4 w-4"
                                                    />
                                                    Delete
                                                </ContextMenuItem>
                                            </ContextMenuContent>
                                        </ContextMenu>
                                    </TreeItem>
                                </TreeRoot>
                            </div>
                        </ScrollArea>
                    </div>
                </ResizablePanel>

                <ResizableHandle with-handle />

                <!-- Middle Panel (Main Content) -->
                <ResizablePanel
                    :default-size="showRightSidebar ? 60 : 80"
                    :min-size="30"
                >
                    <div class="h-full flex flex-col">
                        <div
                            class="flex-grow flex items-center justify-center bg-zinc-50/10 dark:bg-zinc-900/10"
                        >
                            <div class="text-center">
                                <FileText
                                    class="h-16 w-16 mx-auto text-zinc-400 mb-4"
                                />
                                <h3
                                    class="text-xl font-medium text-zinc-600 dark:text-zinc-400 mb-2"
                                >
                                    ProAppLayout
                                </h3>
                                <p
                                    class="text-sm text-zinc-500 dark:text-zinc-500 mb-4"
                                >
                                    A blueprint app layout with three panels
                                </p>
                                <div
                                    class="flex flex-col gap-2 text-xs text-zinc-500"
                                >
                                    <div>
                                        Left Panel:
                                        {{
                                            showLeftSidebar
                                                ? "Visible"
                                                : "Hidden"
                                        }}
                                    </div>
                                    <div>
                                        Right Panel:
                                        {{
                                            showRightSidebar
                                                ? "Visible"
                                                : "Hidden"
                                        }}
                                    </div>
                                    <div>Zoom Level: {{ zoomLevel }}%</div>
                                    <div>
                                        Theme: {{ dark ? "Dark" : "Light" }}
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </ResizablePanel>

                <ResizableHandle with-handle />

                <!-- Right Sidebar Panel -->
                <ResizablePanel
                    ref="rightSidebarPanel"
                    :default-size="showRightSidebar ? 20 : 0"
                    :min-size="15"
                    :collapsed-size="0"
                    collapsible
                >
                    <div
                        class="h-full flex flex-col border-l-1 bg-white/40 dark:bg-zinc-900/40"
                    >
                        <div class="sidebar-header p-3 border-b">
                            <h3 class="text-sm font-semibold">Properties</h3>
                        </div>
                        <ScrollArea class="flex-grow">
                            <div class="p-4 space-y-4">
                                <div>
                                    <Label
                                        class="text-xs font-medium text-zinc-600 dark:text-zinc-300"
                                        >Selected Item</Label
                                    >
                                    <div
                                        class="mt-1 p-2 bg-zinc-100 dark:bg-zinc-800 rounded text-xs"
                                    >
                                        {{
                                            selectedTreeItem?.value?.name ||
                                            "None"
                                        }}
                                    </div>
                                </div>

                                <div>
                                    <Label
                                        class="text-xs font-medium text-zinc-600 dark:text-zinc-300"
                                        >Type</Label
                                    >
                                    <div
                                        class="mt-1 p-2 bg-zinc-100 dark:bg-zinc-800 rounded text-xs"
                                    >
                                        {{
                                            selectedTreeItem?.value?.type ||
                                            "N/A"
                                        }}
                                    </div>
                                </div>

                                <div>
                                    <Label
                                        class="text-xs font-medium text-zinc-600 dark:text-zinc-300"
                                        >Actions</Label
                                    >
                                    <div class="mt-1 flex flex-col gap-1">
                                        <Button
                                            size="sm"
                                            variant="outline"
                                            @click="createNewItem"
                                        >
                                            <Plus class="h-3 w-3 mr-1" />
                                            New Item
                                        </Button>
                                        <Button
                                            size="sm"
                                            variant="outline"
                                            @click="refreshTree"
                                        >
                                            <RefreshCw class="h-3 w-3 mr-1" />
                                            Refresh
                                        </Button>
                                    </div>
                                </div>
                            </div>
                        </ScrollArea>
                    </div>
                </ResizablePanel>
            </ResizablePanelGroup>
        </div>

        <!-- Status Bar -->
        <div
            class="status-bar h-6 bg-white/30 dark:bg-zinc-900/30 text-black dark:text-white text-xs flex items-center justify-between px-3 border-t shrink-0"
        >
            <div class="flex items-center space-x-4">
                <span>ProAppLayout</span>
                <span class="text-zinc-400">Ready</span>
            </div>
            <div class="flex items-center space-x-4">
                <span>{{ treeItems.length }} items</span>
                <span>Zoom {{ zoomLevel }}%</span>
                <span>{{ dark ? "Dark" : "Light" }} Mode</span>
            </div>
        </div>

        <!-- New Item Modal -->
        <Dialog v-model:open="isNewItemModalOpen">
            <DialogContent>
                <DialogHeader>
                    <DialogTitle>Create New Item</DialogTitle>
                    <DialogDescription
                        >Enter the name for the new item</DialogDescription
                    >
                </DialogHeader>
                <div class="space-y-4">
                    <div>
                        <Label for="itemName">Item Name</Label>
                        <Input
                            id="itemName"
                            v-model="newItemName"
                            placeholder="Enter item name"
                            @keydown.enter="confirmNewItem"
                        />
                    </div>
                    <div>
                        <Label for="itemType">Item Type</Label>
                        <select
                            id="itemType"
                            v-model="newItemType"
                            class="w-full p-2 border rounded"
                        >
                            <option value="item">Item</option>
                            <option value="folder">Folder</option>
                        </select>
                    </div>
                </div>
                <DialogFooter>
                    <Button
                        variant="outline"
                        @click="isNewItemModalOpen = false"
                        >Cancel</Button
                    >
                    <Button
                        @click="confirmNewItem"
                        :disabled="!newItemName.trim()"
                        >Create</Button
                    >
                </DialogFooter>
            </DialogContent>
        </Dialog>

        <!-- Delete Confirmation Modal -->
        <Dialog v-model:open="isDeleteModalOpen">
            <DialogContent>
                <DialogHeader>
                    <DialogTitle>Confirm Delete</DialogTitle>
                    <DialogDescription>
                        Are you sure you want to delete "{{
                            selectedTreeItem?.value?.name
                        }}"? This action cannot be undone.
                    </DialogDescription>
                </DialogHeader>
                <DialogFooter>
                    <Button variant="outline" @click="isDeleteModalOpen = false"
                        >Cancel</Button
                    >
                    <Button variant="destructive" @click="confirmDelete"
                        >Delete</Button
                    >
                </DialogFooter>
            </DialogContent>
        </Dialog>

        <!-- Shortcuts Help Modal -->
        <Dialog v-model:open="isShortcutsModalOpen">
            <DialogContent class="max-w-2xl">
                <DialogHeader>
                    <DialogTitle>Keyboard Shortcuts</DialogTitle>
                    <DialogDescription
                        >Available keyboard shortcuts</DialogDescription
                    >
                </DialogHeader>
                <div class="max-h-96 overflow-y-auto">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <div>
                            <h4 class="font-semibold mb-2">General</h4>
                            <div class="space-y-1 text-sm">
                                <div class="flex justify-between">
                                    <span>New Item</span>
                                    <kbd class="bg-muted px-2 py-1 rounded"
                                        >Ctrl+N</kbd
                                    >
                                </div>
                                <div class="flex justify-between">
                                    <span>Open Folder</span>
                                    <kbd class="bg-muted px-2 py-1 rounded"
                                        >Ctrl+O</kbd
                                    >
                                </div>
                                <div class="flex justify-between">
                                    <span>Save</span>
                                    <kbd class="bg-muted px-2 py-1 rounded"
                                        >Ctrl+S</kbd
                                    >
                                </div>
                            </div>
                        </div>
                        <div>
                            <h4 class="font-semibold mb-2">View</h4>
                            <div class="space-y-1 text-sm">
                                <div class="flex justify-between">
                                    <span>Toggle Left Panel</span>
                                    <kbd class="bg-muted px-2 py-1 rounded"
                                        >Ctrl+B</kbd
                                    >
                                </div>
                                <div class="flex justify-between">
                                    <span>Toggle Right Panel</span>
                                    <kbd class="bg-muted px-2 py-1 rounded"
                                        >Ctrl+Shift+B</kbd
                                    >
                                </div>
                                <div class="flex justify-between">
                                    <span>Zoom In</span>
                                    <kbd class="bg-muted px-2 py-1 rounded"
                                        >Ctrl+=</kbd
                                    >
                                </div>
                                <div class="flex justify-between">
                                    <span>Zoom Out</span>
                                    <kbd class="bg-muted px-2 py-1 rounded"
                                        >Ctrl+-</kbd
                                    >
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <DialogFooter>
                    <Button @click="isShortcutsModalOpen = false">Close</Button>
                </DialogFooter>
            </DialogContent>
        </Dialog>

        <!-- Open Folder Dialog -->
        <OpenFolderModal
            :open="isOpenFolderModalOpen"
            @update:open="isOpenFolderModalOpen = $event"
            title="Select Folder"
            description="Choose a folder to open"
            :fs="fs"
            :initial-path="currentPath || '/'"
            @selected="handleFolderOpen"
            @cancel="isOpenFolderModalOpen = false"
        />
    </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, nextTick } from "vue";
import { TreeItem, TreeRoot } from "radix-vue";
import { toast } from "vue-sonner";
import { useDark } from "@vueuse/core";
import { fs } from "@/composables/useFileSystem";
import useSettings from "@/composables/useSettings";
import { useShortcut } from "@/composables/useShortcut";

// Icons
import {
    FileText,
    FolderPlus,
    FolderOpen,
    Save,
    X,
    Check,
    RefreshCw,
    Plus,
    Undo,
    Redo,
    MousePointer,
    ZoomIn,
    ZoomOut,
    RotateCcw,
    Keyboard,
    Folder,
    Edit,
    Trash2,
    ChevronRight,
    PanelLeft,
    PanelRight,
} from "lucide-vue-next";

import OpenFolderModal from "@/components/gui/OpenFolderModal";

// Props
const props = defineProps({
    path: {
        type: String,
        required: false,
        default: "/",
    },
});

// Composables
const dark = useDark();
const { getSetting, setSetting, registerApp } = useSettings();
const { registerLocalShortcut } = useShortcut("ProAppLayout");

// Register ProAppLayout's settings
registerApp("proAppLayout", "ProApp Layout", [
    {
        id: "proapp-layout",
        name: "Layout",
        description: "Configure the ProApp layout behavior",
        icon: FileText,
        order: 1,
        settings: [
            {
                key: "proAppShowLeftSidebar",
                label: "Show Left Panel",
                description: "Display the left sidebar panel",
                type: "boolean",
                default: true,
            },
            {
                key: "proAppShowRightSidebar",
                label: "Show Right Panel",
                description: "Display the right sidebar panel",
                type: "boolean",
                default: false,
            },
            {
                key: "proAppZoomLevel",
                label: "Zoom Level",
                description: "Application zoom level percentage",
                type: "range",
                default: 100,
                min: 50,
                max: 200,
                step: 10,
            },
        ],
    },
]);

// State
const treeItems = ref([
    {
        id: "1",
        name: "Root Folder",
        type: "folder",
        children: [
            { id: "1-1", name: "Document 1", type: "item" },
            { id: "1-2", name: "Document 2", type: "item" },
            {
                id: "1-3",
                name: "Sub Folder",
                type: "folder",
                children: [
                    { id: "1-3-1", name: "Sub Document 1", type: "item" },
                    { id: "1-3-2", name: "Sub Document 2", type: "item" },
                ],
            },
        ],
    },
    { id: "2", name: "Document 3", type: "item" },
    { id: "3", name: "Document 4", type: "item" },
]);

const selectedItemId = ref("");
const selectedTreeItem = ref(null);
const expandedItems = ref(["1"]);
const currentPath = ref(props.path || "/");

// Panel refs
const leftSidebarPanel = ref(null);
const rightSidebarPanel = ref(null);

// Sidebar states (using dynamic settings)
const showLeftSidebar = ref(getSetting("proAppShowLeftSidebar") ?? true);
const showRightSidebar = ref(getSetting("proAppShowRightSidebar") ?? false);

// Zoom level
const zoomLevel = computed({
    get: () => getSetting("proAppZoomLevel") ?? 100,
    set: (value) => setSetting("proAppZoomLevel", value),
});

// Modal states
const isNewItemModalOpen = ref(false);
const isDeleteModalOpen = ref(false);
const isOpenFolderModalOpen = ref(false);
const isShortcutsModalOpen = ref(false);
const contextMenuOpen = ref(false);

// Form inputs
const newItemName = ref("");
const newItemType = ref("item");

// Handle layout changes for main horizontal panels (left sidebar, main, right sidebar)
function handleMainLayoutChange(layout) {
    // layout[0] = left sidebar, layout[1] = main content, layout[2] = right sidebar
    const leftCollapsed = layout[0] <= 1; // Consider collapsed if size <= 1%
    const rightCollapsed = layout[2] <= 1;

    showLeftSidebar.value = !leftCollapsed;
    showRightSidebar.value = !rightCollapsed;

    setSetting("proAppShowLeftSidebar", !leftCollapsed);
    setSetting("proAppShowRightSidebar", !rightCollapsed);
}

// Tree operations
function refreshTree() {
    // Simulate refresh
    toast.success("Tree refreshed");
}

function handleItemClick(item) {
    selectedTreeItem.value = item;
    selectedItemId.value = item.value.id;
}

function openContextMenu(event, item) {
    selectedTreeItem.value = item;
    selectedItemId.value = item.value.id;
    contextMenuOpen.value = true;
}

function updateSelection(newSelection) {
    if (newSelection.length > 0) {
        selectedItemId.value = newSelection[0];
    }
}

// CRUD operations
function createNewItem() {
    newItemName.value = "";
    newItemType.value = "item";
    isNewItemModalOpen.value = true;
}

function createNewFolder() {
    newItemName.value = "";
    newItemType.value = "folder";
    isNewItemModalOpen.value = true;
}

function confirmNewItem() {
    if (!newItemName.value.trim()) return;

    const newItem = {
        id: `${Date.now()}-${Math.random()}`,
        name: newItemName.value,
        type: newItemType.value,
        children: newItemType.value === "folder" ? [] : undefined,
    };

    treeItems.value.push(newItem);
    isNewItemModalOpen.value = false;
    newItemName.value = "";
    toast.success(
        `${newItemType.value === "folder" ? "Folder" : "Item"} created successfully`,
    );
}

function startRename() {
    if (!selectedTreeItem.value) return;
    toast.info("Rename functionality would be implemented here");
    contextMenuOpen.value = false;
}

function deleteSelected() {
    if (!selectedTreeItem.value) return;
    isDeleteModalOpen.value = true;
    contextMenuOpen.value = false;
}

function confirmDelete() {
    if (!selectedTreeItem.value) return;

    // Simple delete from root level (in a real app, you'd traverse the tree)
    const index = treeItems.value.findIndex(
        (item) => item.id === selectedTreeItem.value.value.id,
    );
    if (index !== -1) {
        treeItems.value.splice(index, 1);
        toast.success("Item deleted successfully");
    }

    isDeleteModalOpen.value = false;
    selectedTreeItem.value = null;
    selectedItemId.value = "";
}

// View operations
function toggleLeftSidebar() {
    if (leftSidebarPanel.value) {
        if (leftSidebarPanel.value.isCollapsed) {
            leftSidebarPanel.value.expand();
        } else {
            leftSidebarPanel.value.collapse();
        }
    }
}

function toggleRightSidebar() {
    if (rightSidebarPanel.value) {
        if (rightSidebarPanel.value.isCollapsed) {
            rightSidebarPanel.value.expand();
        } else {
            rightSidebarPanel.value.collapse();
        }
    }
}

// Zoom operations
function zoomIn() {
    zoomLevel.value = Math.min(zoomLevel.value + 10, 200);
    toast.success(`Zoom: ${zoomLevel.value}%`);
}

function zoomOut() {
    zoomLevel.value = Math.max(zoomLevel.value - 10, 50);
    toast.success(`Zoom: ${zoomLevel.value}%`);
}

function resetZoom() {
    zoomLevel.value = 100;
    toast.success("Zoom reset to 100%");
}

// File operations
function openFolder() {
    isOpenFolderModalOpen.value = true;
}

function handleFolderOpen(path) {
    currentPath.value = path;
    isOpenFolderModalOpen.value = false;
    toast.success(`Opened folder: ${path}`);
}

function saveItem() {
    toast.success("Item saved successfully");
}

// Edit operations
function undo() {
    toast.info("Undo operation");
}

function redo() {
    toast.info("Redo operation");
}

function selectAll() {
    toast.info("Select all operation");
}

// Help operations
function showShortcutsDialog() {
    isShortcutsModalOpen.value = true;
}

// Register keyboard shortcuts
const registerProAppShortcuts = () => {
    // File operations
    registerLocalShortcut("ctrl+n", createNewItem);
    registerLocalShortcut("ctrl+o", openFolder);
    registerLocalShortcut("ctrl+s", saveItem);

    // Edit operations
    registerLocalShortcut("ctrl+z", undo);
    registerLocalShortcut("ctrl+y", redo);
    registerLocalShortcut("ctrl+a", selectAll);

    // View operations
    registerLocalShortcut("ctrl+b", toggleLeftSidebar);
    registerLocalShortcut("ctrl+shift+b", toggleRightSidebar);
    registerLocalShortcut("ctrl+=", zoomIn);
    registerLocalShortcut("ctrl+-", zoomOut);
    registerLocalShortcut("ctrl+0", resetZoom);

    // Help
    registerLocalShortcut("ctrl+k ctrl+s", showShortcutsDialog);
};

// Lifecycle hooks
onMounted(() => {
    registerProAppShortcuts();

    nextTick(() => {
        // Initialize panels based on settings
        if (!showLeftSidebar.value && leftSidebarPanel.value) {
            leftSidebarPanel.value.collapse();
        }
        if (!showRightSidebar.value && rightSidebarPanel.value) {
            rightSidebarPanel.value.collapse();
        }
    });
});
</script>

<style scoped>
.sidebar-header {
    background: rgba(0, 0, 0, 0.05);
    backdrop-filter: blur(10px);
    border-bottom: 1px solid rgba(0, 0, 0, 0.1);
}

.dark .sidebar-header {
    background: rgba(255, 255, 255, 0.05);
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.status-bar {
    transition: all 0.2s ease;
}

.tree-item {
    position: relative;
    transition: all 0.2s ease;
}

@media (max-width: 768px) {
    .status-bar {
        font-size: 10px;
        padding: 0 8px;
    }
}

@media (max-width: 640px) {
    .status-bar {
        font-size: 9px;
        padding: 0 6px;
        height: 20px;
    }

    .menubar-trigger {
        padding: 0.25rem 0.5rem;
        font-size: 0.75rem;
    }
}
</style>
