<!-- @app {"title":"Nuklear Browser","icon":"icons/apps/web-browser.svg","width":1200,"height":800,"singleWindow":false} -->
<template>
    <TooltipProvider>
        <div
            class="flex flex-col h-full text-foreground rounded-lg overflow-hidden relative"
        >
            <!-- Chrome-like Header with Tabs and Menu -->
            <div class="flex items-center border-b shrink-0">
                <!-- Tab Bar -->
                <div class="flex-1 flex items-center min-w-0 overflow-hidden">
                    <div class="tab-container flex items-center flex-1 min-w-0">
                        <div
                            v-for="(tab, index) in openTabs"
                            :key="tab.id"
                            :class="[
                                'tab-item pointer-events-auto relative flex items-center gap-2 px-3 py-2 border-r border-border cursor-pointer transition-colors min-w-0 group',
                                {
                                    'bg-background text-foreground':
                                        activeTabIndex === index,
                                    'bg-muted/50 text-muted-foreground hover:bg-muted':
                                        activeTabIndex !== index,
                                },
                            ]"
                            @click="switchToTab(index)"
                            @contextmenu.prevent="
                                openTabContextMenu($event, index)
                            "
                        >
                            <img
                                :src="tab.favicon || defaultFaviconUrl"
                                @error="setDefaultFavicon($event, tab)"
                                class="w-4 h-4 flex-shrink-0"
                                alt=""
                            />
                            <span
                                class="truncate text-sm flex-1 min-w-0"
                                :title="tab.title || tab.url"
                            >
                                {{ tab.title || "New Tab" }}
                            </span>
                            <span
                                v-if="tab.isModified"
                                class="modified-indicator flex-shrink-0"
                                >●</span
                            >
                            <Button
                                @click.stop="closeTab(index)"
                                variant="ghost"
                                size="icon"
                                class="h-4 w-4 ml-1 hover:bg-destructive/20 hover:text-destructive text-muted-foreground transition-all duration-200 group-hover:opacity-100 opacity-60 flex-shrink-0"
                            >
                                <X class="h-3 w-3" />
                            </Button>
                        </div>
                    </div>
                    <Button
                        @click="addNewTab()"
                        variant="ghost"
                        size="icon"
                        class="h-8 w-8 ml-1 flex-shrink-0 pointer-events-auto"
                    >
                        <Plus class="h-4 w-4" />
                    </Button>
                </div>

                <!-- Chrome-like Menu -->
                <div class="flex items-center mr-16 p-2 pointer-events-auto">
                    <DropdownMenu>
                        <DropdownMenuTrigger as-child>
                            <Button variant="ghost" size="icon" class="h-4 w-4">
                                <MoreVertical class="h-4 w-4" />
                            </Button>
                        </DropdownMenuTrigger>
                        <DropdownMenuContent align="end" class="w-56">
                            <DropdownMenuItem @click="addNewTab()">
                                <Plus class="mr-2 h-4 w-4" />
                                New Tab
                            </DropdownMenuItem>
                            <DropdownMenuItem
                                @click="showBookmarksManager = true"
                            >
                                <Star class="mr-2 h-4 w-4" />
                                Bookmarks Manager
                            </DropdownMenuItem>
                            <DropdownMenuSeparator />
                            <DropdownMenuItem @click="showHistory = true">
                                <History class="mr-2 h-4 w-4" />
                                History
                            </DropdownMenuItem>
                            <DropdownMenuItem @click="showDownloads = true">
                                <Download class="mr-2 h-4 w-4" />
                                Downloads
                            </DropdownMenuItem>
                            <DropdownMenuSeparator />
                            <DropdownMenuItem @click="toggleLeftSidebar()">
                                <PanelLeft class="mr-2 h-4 w-4" />
                                {{ showLeftSidebar ? "Hide" : "Show" }} Sidebar
                            </DropdownMenuItem>
                            <DropdownMenuItem @click="showSettings = true">
                                <Settings class="mr-2 h-4 w-4" />
                                Settings
                            </DropdownMenuItem>
                            <DropdownMenuSeparator />
                        </DropdownMenuContent>
                    </DropdownMenu>
                </div>
            </div>

            <!-- Main Content Area with Resizable Panels -->
            <ResizablePanelGroup
                direction="horizontal"
                class="flex-1 min-h-0 pointer-events-auto"
            >
                <!-- Left Sidebar -->
                <ResizablePanel
                    ref="leftSidebarPanel"
                    :default-size="20"
                    :min-size="15"
                    :collapsed-size="0"
                    :max-size="40"
                    collapsible
                >
                    <div class="h-full border-r bg-background/30 flex flex-col">
                        <div class="p-2 border-b">
                            <h3 class="font-semibold text-sm">Navigation</h3>
                        </div>

                        <!-- Bookmarks Panel -->
                        <div class="flex-1 overflow-hidden flex flex-col">
                            <div class="p-2 border-b">
                                <div
                                    class="flex items-center justify-between mb-2"
                                >
                                    <h4 class="font-medium text-sm">
                                        Bookmarks
                                    </h4>
                                    <Button
                                        @click="showAddBookmarkModal = true"
                                        variant="ghost"
                                        size="icon"
                                        class="h-6 w-6"
                                    >
                                        <Plus class="h-3 w-3" />
                                    </Button>
                                </div>
                                <ScrollArea class="h-32">
                                    <div class="space-y-1">
                                        <div
                                            v-for="(
                                                bookmark, index
                                            ) in bookmarks"
                                            :key="index"
                                            class="flex items-center gap-2 p-1 rounded hover:bg-muted cursor-pointer text-sm"
                                            @click="
                                                navigateToBookmark(bookmark)
                                            "
                                            @contextmenu.prevent="
                                                openBookmarkContextMenu(
                                                    $event,
                                                    index,
                                                )
                                            "
                                        >
                                            <img
                                                :src="
                                                    bookmark.favicon ||
                                                    defaultFaviconUrl
                                                "
                                                class="w-3 h-3"
                                                alt=""
                                            />
                                            <span
                                                class="truncate"
                                                :title="bookmark.title"
                                                >{{ bookmark.title }}</span
                                            >
                                        </div>
                                    </div>
                                </ScrollArea>
                            </div>

                            <!-- History Panel -->
                            <div class="p-2 flex-1 overflow-hidden">
                                <h4 class="font-medium text-sm mb-2">
                                    Recent History
                                </h4>
                                <ScrollArea class="h-full">
                                    <div class="space-y-1">
                                        <div
                                            v-for="(
                                                item, index
                                            ) in recentHistory.slice(0, 20)"
                                            :key="index"
                                            class="flex items-center gap-2 p-1 rounded hover:bg-muted cursor-pointer text-sm"
                                            @click="navigateToUrl(item.url)"
                                        >
                                            <img
                                                :src="
                                                    item.favicon ||
                                                    defaultFaviconUrl
                                                "
                                                class="w-3 h-3"
                                                alt=""
                                            />
                                            <span
                                                class="truncate"
                                                :title="item.title"
                                                >{{
                                                    item.title || item.url
                                                }}</span
                                            >
                                        </div>
                                    </div>
                                </ScrollArea>
                            </div>
                        </div>
                    </div>
                </ResizablePanel>

                <ResizableHandle with-handle />

                <!-- Center Panel - Browser View -->
                <ResizablePanel
                    :default-size="showLeftSidebar ? 80 : 100"
                    :min-size="50"
                >
                    <div class="h-full flex flex-col" v-if="activeTab">
                        <!-- Address Bar -->
                        <div
                            class="flex items-center gap-1 px-2 py-1 border-b bg-background/80"
                        >
                            <Button
                                @click="goBack()"
                                :disabled="!canGoBack"
                                variant="outline"
                                size="icon"
                                class="h-7 w-7"
                            >
                                <ArrowLeft class="h-3.5 w-3.5" />
                            </Button>
                            <Button
                                @click="goForward()"
                                :disabled="!canGoForward"
                                variant="outline"
                                size="icon"
                                class="h-7 w-7"
                            >
                                <ArrowRight class="h-3.5 w-3.5" />
                            </Button>
                            <Button
                                @click="reloadPage()"
                                variant="outline"
                                size="icon"
                                class="h-7 w-7"
                            >
                                <RefreshCw class="h-3.5 w-3.5" />
                            </Button>

                            <div class="flex-1 flex items-center gap-1">
                                <Input
                                    ref="addressBarInput"
                                    v-model="addressBarUrl"
                                    @keyup.enter="handleAddressBarSubmit"
                                    @focus="handleAddressBarFocus"
                                    placeholder="Enter URL or search..."
                                    class="flex-1 h-7"
                                />
                                <Button
                                    @click="toggleBookmarkCurrentPage()"
                                    variant="outline"
                                    size="icon"
                                    class="h-7 w-7"
                                >
                                    <Star
                                        :class="[
                                            'h-3.5 w-3.5',
                                            isCurrentPageBookmarked
                                                ? 'fill-yellow-400 text-yellow-500'
                                                : '',
                                        ]"
                                    />
                                </Button>
                            </div>
                        </div>

                        <!-- Browser Content -->
                        <div class="relative bg-background flex-1 min-h-0">
                            <div
                                v-if="activeTab.isLoading"
                                class="absolute inset-0 flex items-center justify-center bg-background/80"
                            >
                                <div class="flex items-center gap-2">
                                    <div
                                        class="animate-spin rounded-full h-6 w-6 border-b-2 border-primary"
                                    ></div>
                                    <span class="text-sm">Loading...</span>
                                </div>
                            </div>

                            <div
                                v-if="activeTab.error"
                                class="absolute inset-0 flex items-center justify-center"
                            >
                                <div class="text-center">
                                    <AlertTriangle
                                        class="h-12 w-12 text-destructive mx-auto mb-4"
                                    />
                                    <h3 class="text-lg font-semibold mb-2">
                                        Failed to load page
                                    </h3>
                                    <p
                                        class="text-sm text-muted-foreground mb-4"
                                    >
                                        {{ activeTab.url }}
                                    </p>
                                    <Button
                                        @click="reloadPage()"
                                        variant="outline"
                                        >Try Again</Button
                                    >
                                </div>
                            </div>

                            <iframe
                                v-show="
                                    !activeTab.error &&
                                    activeTab.url &&
                                    activeTab.url !== 'about:blank'
                                "
                                :src="activeTab.url"
                                @load="onIframeLoad()"
                                @error="onIframeError()"
                                class="w-full h-full border-0"
                                sandbox="allow-forms allow-modals allow-pointer-lock allow-popups allow-popups-to-escape-sandbox allow-presentation allow-same-origin allow-scripts allow-top-navigation allow-top-navigation-by-user-activation"
                                allow="
                                    encrypted-media;
                                    geolocation;
                                    microphone;
                                    camera;
                                    midi;
                                    display-capture;
                                    payment;
                                "
                            ></iframe>

                            <div
                                v-if="
                                    !activeTab.url ||
                                    activeTab.url === 'about:blank'
                                "
                                class="absolute inset-0 flex items-center justify-center"
                            >
                                <div class="text-center">
                                    <Globe
                                        class="h-16 w-16 text-muted-foreground/50 mx-auto mb-4"
                                    />
                                    <h3 class="text-lg font-semibold mb-2">
                                        New Tab
                                    </h3>
                                    <p class="text-sm text-muted-foreground">
                                        Enter a URL above to start browsing
                                    </p>
                                </div>
                            </div>
                        </div>
                    </div>
                </ResizablePanel>
            </ResizablePanelGroup>

            <!-- Modals -->

            <!-- Add Bookmark Modal -->
            <Dialog v-model:open="showAddBookmarkModal">
                <DialogContent>
                    <DialogHeader>
                        <DialogTitle>Add Bookmark</DialogTitle>
                    </DialogHeader>
                    <div class="space-y-4">
                        <div>
                            <Label for="bookmark-title">Title</Label>
                            <Input
                                id="bookmark-title"
                                v-model="newBookmark.title"
                            />
                        </div>
                        <div>
                            <Label for="bookmark-url">URL</Label>
                            <Input
                                id="bookmark-url"
                                v-model="newBookmark.url"
                            />
                        </div>
                    </div>
                    <DialogFooter>
                        <Button
                            @click="showAddBookmarkModal = false"
                            variant="outline"
                            >Cancel</Button
                        >
                        <Button @click="addBookmark()">Add Bookmark</Button>
                    </DialogFooter>
                </DialogContent>
            </Dialog>

            <!-- Bookmarks Manager Modal -->
            <Dialog v-model:open="showBookmarksManager">
                <DialogContent class="max-w-2xl">
                    <DialogHeader>
                        <DialogTitle>Bookmarks Manager</DialogTitle>
                    </DialogHeader>
                    <div class="space-y-4">
                        <div class="flex items-center justify-between">
                            <span class="text-sm text-muted-foreground"
                                >{{ bookmarks.length }} bookmarks</span
                            >
                            <Button
                                @click="showAddBookmarkModal = true"
                                size="sm"
                            >
                                <Plus class="h-4 w-4 mr-2" />
                                Add Bookmark
                            </Button>
                        </div>
                        <ScrollArea class="h-96">
                            <div class="space-y-2">
                                <div
                                    v-for="(bookmark, index) in bookmarks"
                                    :key="index"
                                    class="flex items-center justify-between p-3 border rounded-lg hover:bg-muted/50"
                                >
                                    <div
                                        class="flex items-center gap-3 flex-1 min-w-0"
                                    >
                                        <img
                                            :src="
                                                bookmark.favicon ||
                                                defaultFaviconUrl
                                            "
                                            class="w-4 h-4 flex-shrink-0"
                                            alt=""
                                        />
                                        <div class="flex-1 min-w-0">
                                            <div class="font-medium truncate">
                                                {{ bookmark.title }}
                                            </div>
                                            <div
                                                class="text-sm text-muted-foreground truncate"
                                            >
                                                {{ bookmark.url }}
                                            </div>
                                        </div>
                                    </div>
                                    <div class="flex items-center gap-2">
                                        <Button
                                            @click="
                                                (navigateToBookmark(bookmark),
                                                (showBookmarksManager = false))
                                            "
                                            size="sm"
                                            variant="outline"
                                        >
                                            Open
                                        </Button>
                                        <Button
                                            @click="deleteBookmark(index)"
                                            size="sm"
                                            variant="destructive"
                                        >
                                            <X class="h-3 w-3" />
                                        </Button>
                                    </div>
                                </div>
                            </div>
                        </ScrollArea>
                    </div>
                </DialogContent>
            </Dialog>

            <!-- History Modal -->
            <Dialog v-model:open="showHistory">
                <DialogContent class="max-w-2xl">
                    <DialogHeader>
                        <DialogTitle>Browsing History</DialogTitle>
                    </DialogHeader>
                    <div class="space-y-4">
                        <div class="flex items-center justify-between">
                            <span class="text-sm text-muted-foreground"
                                >{{ recentHistory.length }} items</span
                            >
                            <Button
                                @click="clearHistory()"
                                size="sm"
                                variant="outline"
                            >
                                Clear History
                            </Button>
                        </div>
                        <ScrollArea class="h-96">
                            <div class="space-y-2">
                                <div
                                    v-for="(item, index) in recentHistory"
                                    :key="index"
                                    class="flex items-center justify-between p-3 border rounded-lg hover:bg-muted/50"
                                >
                                    <div
                                        class="flex items-center gap-3 flex-1 min-w-0"
                                    >
                                        <img
                                            :src="
                                                item.favicon ||
                                                defaultFaviconUrl
                                            "
                                            class="w-4 h-4 flex-shrink-0"
                                            alt=""
                                        />
                                        <div class="flex-1 min-w-0">
                                            <div class="font-medium truncate">
                                                {{ item.title || item.url }}
                                            </div>
                                            <div
                                                class="text-sm text-muted-foreground truncate"
                                            >
                                                {{ item.url }}
                                            </div>
                                            <div
                                                class="text-xs text-muted-foreground"
                                            >
                                                {{
                                                    formatTimestamp(
                                                        item.timestamp,
                                                    )
                                                }}
                                            </div>
                                        </div>
                                    </div>
                                    <div class="flex items-center gap-2">
                                        <Button
                                            @click="
                                                (navigateToUrl(item.url),
                                                (showHistory = false))
                                            "
                                            size="sm"
                                            variant="outline"
                                        >
                                            Visit
                                        </Button>
                                        <Button
                                            @click="removeFromHistory(index)"
                                            size="sm"
                                            variant="destructive"
                                        >
                                            <X class="h-3 w-3" />
                                        </Button>
                                    </div>
                                </div>
                            </div>
                        </ScrollArea>
                    </div>
                </DialogContent>
            </Dialog>

            <!-- Downloads Modal -->
            <Dialog v-model:open="showDownloads">
                <DialogContent class="max-w-2xl">
                    <DialogHeader>
                        <DialogTitle>Downloads</DialogTitle>
                    </DialogHeader>
                    <div class="flex items-center justify-center h-32">
                        <div class="text-center text-muted-foreground">
                            <Download class="h-8 w-8 mx-auto mb-2" />
                            <p>Download functionality not yet implemented</p>
                            <p class="text-sm">
                                Downloads would appear here when available
                            </p>
                        </div>
                    </div>
                </DialogContent>
            </Dialog>

            <!-- Settings Modal -->
            <Dialog v-model:open="showSettings">
                <DialogContent class="max-w-lg">
                    <DialogHeader>
                        <DialogTitle>Nuklear Settings</DialogTitle>
                    </DialogHeader>
                    <div class="space-y-4">
                        <div class="flex items-center justify-between">
                            <div>
                                <div class="font-medium">Show Left Sidebar</div>
                                <div class="text-sm text-muted-foreground">
                                    Display navigation sidebar with bookmarks
                                    and history
                                </div>
                            </div>
                            <Button
                                @click="toggleLeftSidebar()"
                                :variant="
                                    showLeftSidebar ? 'default' : 'outline'
                                "
                                size="sm"
                            >
                                {{ showLeftSidebar ? "On" : "Off" }}
                            </Button>
                        </div>
                    </div>
                </DialogContent>
            </Dialog>

            <!-- Shortcuts Modal -->
            <Dialog v-model:open="showShortcutsModal">
                <DialogContent class="max-w-2xl">
                    <DialogHeader>
                        <DialogTitle>Keyboard Shortcuts</DialogTitle>
                    </DialogHeader>
                    <div class="grid grid-cols-2 gap-4 text-sm">
                        <div>
                            <h4 class="font-semibold mb-2">Navigation</h4>
                            <div class="space-y-1">
                                <div class="flex justify-between">
                                    <span>New Tab</span><kbd>Ctrl+T</kbd>
                                </div>
                                <div class="flex justify-between">
                                    <span>Close Tab</span><kbd>Ctrl+W</kbd>
                                </div>
                                <div class="flex justify-between">
                                    <span>Next Tab</span><kbd>Ctrl+Tab</kbd>
                                </div>
                                <div class="flex justify-between">
                                    <span>Previous Tab</span
                                    ><kbd>Ctrl+Shift+Tab</kbd>
                                </div>
                                <div class="flex justify-between">
                                    <span>Reload</span><kbd>Ctrl+R</kbd>
                                </div>
                                <div class="flex justify-between">
                                    <span>Back</span><kbd>Alt+Left</kbd>
                                </div>
                                <div class="flex justify-between">
                                    <span>Forward</span><kbd>Alt+Right</kbd>
                                </div>
                            </div>
                        </div>
                        <div>
                            <h4 class="font-semibold mb-2">Bookmarks</h4>
                            <div class="space-y-1">
                                <div class="flex justify-between">
                                    <span>Bookmark Page</span><kbd>Ctrl+D</kbd>
                                </div>
                                <div class="flex justify-between">
                                    <span>Bookmarks Manager</span
                                    ><kbd>Ctrl+Shift+O</kbd>
                                </div>
                            </div>
                            <h4 class="font-semibold mb-2 mt-4">General</h4>
                            <div class="space-y-1">
                                <div class="flex justify-between">
                                    <span>Address Bar</span><kbd>Ctrl+L</kbd>
                                </div>
                                <div class="flex justify-between">
                                    <span>Find</span><kbd>Ctrl+F</kbd>
                                </div>
                            </div>
                        </div>
                    </div>
                </DialogContent>
            </Dialog>
        </div>
    </TooltipProvider>
</template>

<script setup>
import { ref, computed, onMounted, nextTick, watch } from 'vue'
import {
  Plus,
  X,
  MoreVertical,
  Star,
  History,
  Download,
  Settings,
  PanelLeft,
  FileText,
  ArrowLeft,
  ArrowRight,
  RefreshCw,
  Globe,
  AlertTriangle
} from 'lucide-vue-next'
import useSettings from '@/composables/useSettings'
import { useShortcut } from '@/composables/useShortcut'

const props = defineProps({
  id: {
    type: String,
    required: true
  }
})

const { getSetting, setSetting, registerApp } = useSettings()
const { registerLocalShortcut } = useShortcut()

// State
const openTabs = ref([])
const activeTabIndex = ref(0)
const addressBarUrl = ref('')
const addressBarInput = ref()
const defaultFaviconUrl =
  'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTYiIGhlaWdodD0iMTYiIHZpZXdCb3g9IjAgMCAxNiAxNiIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTggMS41QzQuNDEgMS41IDEuNSA0LjQxIDEuNSA4UzQuNDEgMTQuNSA4IDE0LjVTMTQuNSAxMS41OSAxNC41IDhTMTEuNTkgMS41IDggMS41WiIgZmlsbD0iY3VycmVudENvbG9yIiBmaWxsLW9wYWNpdHk9IjAuMiIvPgo8L3N2Zz4K'

// Settings
const leftSidebarPanel = ref(null)
const showLeftSidebar = ref(getSetting('nuklearShowLeftSidebar') ?? true)

// Watch panel collapse state and save to settings
watch(
  () => leftSidebarPanel.value?.isCollapsed,
  (isCollapsed) => {
    if (isCollapsed !== undefined) {
      showLeftSidebar.value = !isCollapsed
      setSetting('nuklearShowLeftSidebar', !isCollapsed)
    }
  }
)

// Restore panel state on mount
onMounted(() => {
  nextTick(() => {
if (leftSidebarPanel.value) {
  if (showLeftSidebar.value) {
    leftSidebarPanel.value.expand()
  } else {
    leftSidebarPanel.value.collapse()
  }
}
  })
})
const bookmarks = ref(getSetting('nuklearBookmarks') ?? [])
const recentHistory = ref(getSetting('nuklearHistory') ?? [])

// Modal states
const showAddBookmarkModal = ref(false)

const showShortcutsModal = ref(false)
const showBookmarksManager = ref(false)
const showHistory = ref(false)
const showDownloads = ref(false)
const showSettings = ref(false)

// Form data
const newBookmark = ref({ title: '', url: '' })

// Computed
const activeTab = computed(() => openTabs.value[activeTabIndex.value])

const canGoBack = computed(() => {
  if (!activeTab.value) return false
  return activeTab.value.historyIndex > 0
})

const canGoForward = computed(() => {
  if (!activeTab.value) return false
  return activeTab.value.historyIndex < activeTab.value.history.length - 1
})

const isCurrentPageBookmarked = computed(() => {
  if (!activeTab.value?.url) return false
  return bookmarks.value.some((b) => b.url === activeTab.value?.url)
})

// Functions
function generateId() {
  return Math.random().toString(36).substr(2, 9)
}

function sanitizeUrl(url) {
  if (!url) return 'about:blank'

  // If it looks like a search term, use Google search
  if (!url.includes('.') && !url.startsWith('http')) {
  return `https://www.google.com/search?q=${encodeURIComponent(url)}`
  }

  // Add protocol if missing
  if (!url.startsWith('http://') && !url.startsWith('https://')) {
return `https://${url}`
  }

  return url
}

function getHostname(url) {
  try {
  return new URL(url).hostname
  } catch {
  return url
  }
}

function getFavicon(url) {
  if (!url || url === 'about:blank') return defaultFaviconUrl
  try {
  const domain = new URL(url).hostname
  return `https://www.google.com/s2/favicons?domain=${domain}&sz=16`
  } catch {
  return defaultFaviconUrl
  }
}

function setDefaultFavicon(event, tab) {
  const img = event.target
  img.src = defaultFaviconUrl
  if (tab) {
  tab.favicon = defaultFaviconUrl
  }
}

function addNewTab(url = 'about:blank') {
  const newTab = {
  id: generateId(),
  url: url,
  title: url === 'about:blank' ? 'New Tab' : url,
  favicon: getFavicon(url),
  isLoading: false,
  isModified: false,
  history: [url],
  historyIndex: 0
  }

  openTabs.value.push(newTab)
  activeTabIndex.value = openTabs.value.length - 1
  updateAddressBar()
  saveTabs()
}

function switchToTab(index) {
  if (index >= 0 && index < openTabs.value.length) {
  activeTabIndex.value = index
  updateAddressBar()
  saveTabs()
  }
}

function closeTab(index) {
  if (openTabs.value.length === 1) {
// Always keep at least one tab
  openTabs.value[0] = {
  id: generateId(),
  url: 'about:blank',
  title: 'New Tab',
  favicon: defaultFaviconUrl,
  isLoading: false,
  isModified: false,
  history: ['about:blank'],
  historyIndex: 0
}
  activeTabIndex.value = 0
  addressBarUrl.value = ''
  } else {
  openTabs.value.splice(index, 1)
  if (activeTabIndex.value >= index && activeTabIndex.value > 0) {
  activeTabIndex.value--
}
  if (activeTabIndex.value >= openTabs.value.length) {
  activeTabIndex.value = openTabs.value.length - 1
}
  updateAddressBar()
  }
  saveTabs()
}

function navigateToUrl(url) {
  if (!activeTab.value) return

  const sanitizedUrl = sanitizeUrl(url)
  activeTab.value.url = sanitizedUrl
  activeTab.value.isLoading = true
  activeTab.value.error = undefined

  // Update history
  if (activeTab.value.historyIndex < activeTab.value.history.length - 1) {
// Remove forward history if we're navigating to a new page
  activeTab.value.history = activeTab.value.history.slice(0, activeTab.value.historyIndex + 1)
  }
  activeTab.value.history.push(sanitizedUrl)
  activeTab.value.historyIndex = activeTab.value.history.length - 1

  addressBarUrl.value = sanitizedUrl

  // Add to global history
  addToHistory(sanitizedUrl, sanitizedUrl)

  // Blur the address bar after navigation
  if (addressBarInput.value) {
addressBarInput.value.blur()
  }

  saveTabs()
}

function handleAddressBarSubmit() {
  navigateToUrl(addressBarUrl.value)
}

function handleAddressBarFocus() {
  // Select all text when focusing the address bar for easy editing
  nextTick(() => {
  if (addressBarInput.value) {
  addressBarInput.value.select()
}
  })
}

function focusAddressBar() {
  if (addressBarInput.value) {
  addressBarInput.value.focus()
  addressBarInput.value.select()
  }
}

function navigateToBookmark(bookmark) {
  navigateToUrl(bookmark.url)
}

function goBack() {
  if (!canGoBack.value || !activeTab.value) return

  activeTab.value.historyIndex--
  const url = activeTab.value.history[activeTab.value.historyIndex]
  activeTab.value.url = url
  addressBarUrl.value = url
  activeTab.value.isLoading = true
  saveTabs()
}

function goForward() {
  if (!canGoForward.value || !activeTab.value) return

  activeTab.value.historyIndex++
  const url = activeTab.value.history[activeTab.value.historyIndex]
  activeTab.value.url = url
  addressBarUrl.value = url
  activeTab.value.isLoading = true
  saveTabs()
}

function reloadPage() {
  if (!activeTab.value?.url) return

  activeTab.value.isLoading = true
  activeTab.value.error = undefined

  // Force iframe reload by changing src temporarily
  const currentUrl = activeTab.value.url
  activeTab.value.url = 'about:blank'
  nextTick(() => {
  if (activeTab.value) {
  activeTab.value.url = currentUrl
}
  })
}

function onIframeLoad() {
  if (activeTab.value) {
  activeTab.value.isLoading = false

// Update favicon immediately
  activeTab.value.favicon = getFavicon(activeTab.value.url)

// Enhanced title extraction with multiple fallback strategies
  const attemptTitleExtraction = () => {
  try {
    const iframe = document.querySelector('iframe')
    const doc = iframe?.contentDocument

    if (!doc) return false

    // Strategy 1: Document title
    if (doc.title && doc.title.trim() && doc.title !== 'Document') {
      activeTab.value.title = doc.title.trim()
      saveTabs()
      return true
    }

    // Strategy 2: Meta title or og:title
    const metaTitle = doc.querySelector('meta[property="og:title"]')
    if (metaTitle?.content?.trim()) {
      activeTab.value.title = metaTitle.content.trim()
      saveTabs()
      return true
    }

    // Strategy 3: First h1 tag
    const h1 = doc.querySelector('h1')
    if (h1?.textContent?.trim()) {
      activeTab.value.title = h1.textContent.trim()
      saveTabs()
      return true
    }

    // Strategy 4: First h2 tag if no h1
    const h2 = doc.querySelector('h2')
    if (h2?.textContent?.trim()) {
      activeTab.value.title = h2.textContent.trim()
      saveTabs()
      return true
    }

    // Strategy 5: Meta description as last resort
    const metaDesc = doc.querySelector('meta[name="description"]')
    if (metaDesc?.content?.trim()) {
      const desc = metaDesc.content.trim()
      activeTab.value.title = desc.length > 50 ? desc.substring(0, 47) + '...' : desc
      saveTabs()
      return true
    }
  } catch (e) {
    // CORS prevents access
  }
  return false
}

// Try immediately
  if (!attemptTitleExtraction()) {
  // Fallback to domain-based title
  try {
    const url = new URL(activeTab.value.url)
    // Capitalize and clean up domain name
    activeTab.value.title = url.hostname
      .replace(/^www\./, '')
      .split('.')
      .map((part) => part.charAt(0).toUpperCase() + part.slice(1))
      .join(' ')
  } catch {
    activeTab.value.title = activeTab.value.url
  }

  // Retry after delays to catch dynamically loaded titles
  const retryAttempts = [500, 1500, 3000]
  retryAttempts.forEach((delay, index) => {
    setTimeout(() => {
      if (activeTab.value && activeTab.value.url && !activeTab.value.title.includes('.')) {
        attemptTitleExtraction()
      }
    }, delay)
  })
}

  saveTabs()
  }
}

function onIframeError() {
  if (activeTab.value) {
  activeTab.value.isLoading = false
  activeTab.value.error = 'Failed to load page'
  saveTabs()
  }
}

function toggleBookmarkCurrentPage() {
  if (!activeTab.value?.url || activeTab.value.url === 'about:blank') return

  const existingIndex = bookmarks.value.findIndex((b) => b.url === activeTab.value?.url)

  if (existingIndex >= 0) {
// Remove bookmark
  bookmarks.value.splice(existingIndex, 1)
  } else {
// Add bookmark
  bookmarks.value.push({
  title: activeTab.value.title || activeTab.value.url,
  url: activeTab.value.url,
  favicon: activeTab.value.favicon
})
  }

  saveBookmarks()
}

function addBookmark() {
  if (!newBookmark.value.title || !newBookmark.value.url) return

  bookmarks.value.push({
  title: newBookmark.value.title,
  url: sanitizeUrl(newBookmark.value.url),
  favicon: getFavicon(newBookmark.value.url)
  })

  newBookmark.value = { title: '', url: '' }
  showAddBookmarkModal.value = false
  saveBookmarks()
}

function deleteBookmark(index) {
  bookmarks.value.splice(index, 1)
  saveBookmarks()
}

function clearHistory() {
  recentHistory.value = []
  saveHistory()
}

function removeFromHistory(index) {
  recentHistory.value.splice(index, 1)
  saveHistory()
}

function formatTimestamp(timestamp) {
  const date = new Date(timestamp)
  const now = new Date()
  const diffMs = now.getTime() - date.getTime()
  const diffMins = Math.floor(diffMs / 60000)
  const diffHours = Math.floor(diffMs / 3600000)
  const diffDays = Math.floor(diffMs / 86400000)

  if (diffMins < 1) return 'Just now'
  if (diffMins < 60) return `${diffMins} minutes ago`
  if (diffHours < 24) return `${diffHours} hours ago`
  if (diffDays < 7) return `${diffDays} days ago`
  return date.toLocaleDateString()
}

function addToHistory(url, title) {
  if (url === 'about:blank') return

  const historyItem = {
  url,
  title,
  favicon: getFavicon(url),
  timestamp: Date.now()
  }

  // Remove existing entry for this URL
  const existingIndex = recentHistory.value.findIndex((item) => item.url === url)
  if (existingIndex >= 0) {
recentHistory.value.splice(existingIndex, 1)
  }

  // Add to beginning
  recentHistory.value.unshift(historyItem)

  // Keep only last 100 items
  if (recentHistory.value.length > 100) {
recentHistory.value = recentHistory.value.slice(0, 100)
  }

  saveHistory()
}

function toggleLeftSidebar() {
  if (leftSidebarPanel.value) {
  if (leftSidebarPanel.value.isCollapsed) {
  leftSidebarPanel.value.expand()
} else {
  leftSidebarPanel.value.collapse()
}
  }
}

function saveTabs() {
  setSetting('nuklearTabs', openTabs.value)
  setSetting('nuklearActiveTabIndex', activeTabIndex.value)
}

function loadTabs() {
  const savedTabs = getSetting('nuklearTabs') ?? []
  const savedActiveIndex = getSetting('nuklearActiveTabIndex') ?? 0

  if (savedTabs.length > 0) {
  openTabs.value = savedTabs.map((tab) => ({
  ...tab,
  id: generateId(), // Generate new IDs
  isLoading: false,
  error: undefined,
  favicon: tab.favicon || getFavicon(tab.url)
}))
  activeTabIndex.value = Math.min(savedActiveIndex, openTabs.value.length - 1)
  } else {
  addNewTab()
  }

  // Update address bar with current tab URL
  updateAddressBar()
}

function saveBookmarks() {
  setSetting('nuklearBookmarks', bookmarks.value)
}

function saveHistory() {
  setSetting('nuklearHistory', recentHistory.value)
}

function openTabContextMenu(event, index) {
  // Context menu functionality can be added here
}

function openBookmarkContextMenu(event, index) {
  // Context menu functionality can be added here
}

function switchToNextTab() {
  const nextIndex = (activeTabIndex.value + 1) % openTabs.value.length
  switchToTab(nextIndex)
}

function switchToPreviousTab() {
  const prevIndex =
  activeTabIndex.value === 0 ? openTabs.value.length - 1 : activeTabIndex.value - 1
  switchToTab(prevIndex)
}

function registerNuklearShortcuts() {
  registerLocalShortcut('ctrl+t', () => addNewTab())
  registerLocalShortcut('ctrl+w', () => closeTab(activeTabIndex.value))
  registerLocalShortcut('ctrl+tab', switchToNextTab)
  registerLocalShortcut('ctrl+shift+tab', switchToPreviousTab)
  registerLocalShortcut('ctrl+r', reloadPage)
  registerLocalShortcut('f5', reloadPage)
  registerLocalShortcut('alt+left', goBack)
  registerLocalShortcut('alt+right', goForward)
  registerLocalShortcut('ctrl+d', toggleBookmarkCurrentPage)
  registerLocalShortcut('ctrl+shift+o', () => (showBookmarksManager.value = true))
  registerLocalShortcut('ctrl+h', () => (showHistory.value = true))
  registerLocalShortcut('ctrl+l', focusAddressBar)

  // Tab switching by number
  for (let i = 1; i <= 9; i++) {
  registerLocalShortcut(`ctrl+${i}`, () => {
  if (i <= openTabs.value.length) {
    switchToTab(i - 1)
  }
})
  }
}

function updateAddressBar() {
  if (activeTab.value) {
  addressBarUrl.value = activeTab.value.url === 'about:blank' ? '' : activeTab.value.url
  }
}

// Lifecycle
onMounted(() => {
  loadTabs()
  registerNuklearShortcuts()
})

// Watch for settings changes

watch(
  bookmarks,
  () => {
saveBookmarks()
  },
  { deep: true }
)

watch(
  recentHistory,
  () => {
saveHistory()
  },
  { deep: true }
)

// Register the app
registerApp('nuklear', 'Nuklear Browser', [
  {
    id: 'nuklear-general',
    name: 'General',
    description: 'Configure Nuklear browser behavior',
    icon: Globe,
    order: 1,
    settings: [
      {
        key: 'nuklearShowLeftSidebar',
        label: 'Show Left Sidebar',
        description: 'Show the navigation sidebar with bookmarks and history',
        type: 'boolean',
        default: true
      }
    ]
  }
])

</script>

<style scoped>
.modified-indicator {
    color: hsl(var(--orange-500));
    font-weight: bold;
    font-size: 0.75rem;
    line-height: 1;
    margin-left: 2px;
}

.status-bar {
    font-size: 11px;
}

.tab-container {
    display: flex;
    align-items: center;
    flex: 1;
    min-width: 0;
    overflow: hidden;
}

.tab-item {
    position: relative;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.5rem 0.75rem;
    border-right: 1px solid hsl(var(--border));
    cursor: pointer;
    transition: all 200ms;
    min-width: 0;
    max-width: 200px;
    flex: 1 1 auto;
    overflow: hidden;
}

.tab-item:hover {
    background-color: hsl(var(--muted));
}

.tab-item:hover .close-tab-btn {
    opacity: 1;
}

.tab-item.active {
    background-color: hsl(var(--background));
    color: hsl(var(--foreground));
    border-bottom: 2px solid hsl(var(--primary));
}

.close-tab-btn {
    opacity: 0.6;
    transition: all 200ms;
    flex-shrink: 0;
}

.close-tab-btn:hover {
    opacity: 1;
    background-color: hsl(var(--destructive) / 0.1);
}

@media (max-width: 768px) {
    .tab-item {
        max-width: 120px;
        padding: 0.5rem;
        gap: 0.25rem;
    }

    .status-bar {
        font-size: 0.75rem;
    }
}

@media (max-width: 640px) {
    .tab-item {
        max-width: 80px;
        padding: 0.25rem 0.5rem;
        gap: 0.25rem;
    }

    .tab-item span {
        font-size: 0.75rem;
    }

    .tab-item .close-tab-btn {
        display: none;
    }
}

.tab-item:last-child {
    border-right: none;
}
</style>
