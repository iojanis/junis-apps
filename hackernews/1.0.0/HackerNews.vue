<!-- @app {"title":"HackerNews","icon":"icons/apps/hacker-news.svg","width":1200,"height":720,"singleWindow":false, "inset":true} -->
<template>
    <div class="h-full flex flex-col bg-transparent">
        <Menubar
            @mousedown.stop
            class="border-b shrink-0 bg-white/80 dark:bg-zinc-900/80 backdrop-blur-sm"
        >
            <MenubarMenu class="menubar-menu pointer-events-auto">
                <MenubarTrigger class="menubar-trigger">Stories</MenubarTrigger>
                <MenubarContent>
                    <MenubarItem
                        @click="loadTopStories"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <TrendingUp class="mr-2 h-4 w-4" />
                            Top Stories
                        </div>
                        <MenubarShortcut>Ctrl+T</MenubarShortcut>
                    </MenubarItem>
                    <MenubarItem
                        @click="loadNewStories"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <Clock class="mr-2 h-4 w-4" />
                            New Stories
                        </div>
                        <MenubarShortcut>Ctrl+N</MenubarShortcut>
                    </MenubarItem>
                    <MenubarItem
                        @click="loadBestStories"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <Star class="mr-2 h-4 w-4" />
                            Best Stories
                        </div>
                        <MenubarShortcut>Ctrl+B</MenubarShortcut>
                    </MenubarItem>
                    <MenubarSeparator />
                    <MenubarItem
                        @click="refreshStories"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <RefreshCw class="mr-2 h-4 w-4" />
                            Refresh
                        </div>
                        <MenubarShortcut>F5</MenubarShortcut>
                    </MenubarItem>
                </MenubarContent>
            </MenubarMenu>

            <MenubarMenu class="menubar-menu">
                <MenubarTrigger class="menubar-trigger">View</MenubarTrigger>
                <MenubarContent>
                    <MenubarItem
                        @click="toggleSidebar"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <Check v-if="showSidebar" class="mr-2 h-4 w-4" />
                            <span class="ml-6" v-else />
                            <PanelRight class="mr-2 h-4 w-4" />
                            {{ showSidebar ? "Hide" : "Show" }} Comments
                        </div>
                        <MenubarShortcut>Ctrl+D</MenubarShortcut>
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
                </MenubarContent>
            </MenubarMenu>

            <MenubarMenu class="menubar-menu">
                <MenubarTrigger class="menubar-trigger"
                    >Settings</MenubarTrigger
                >
                <MenubarContent>
                    <MenubarItem
                        @click="toggleDarkMode"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <Moon v-if="!isDarkMode" class="mr-2 h-4 w-4" />
                            <Sun v-else class="mr-2 h-4 w-4" />
                            {{ isDarkMode ? "Light" : "Dark" }} Mode
                        </div>
                    </MenubarItem>
                    <MenubarSeparator />
                    <MenubarItem
                        @click="openSettings"
                        class="flex items-center justify-between"
                    >
                        <div class="flex items-center">
                            <Settings class="mr-2 h-4 w-4" />
                            Preferences
                        </div>
                    </MenubarItem>
                </MenubarContent>
            </MenubarMenu>
        </Menubar>

        <div class="flex-grow pointer-events-auto flex overflow-hidden">
            <ResizablePanelGroup direction="horizontal">
                <!-- Main Content Panel -->
                <ResizablePanel
                    :default-size="showSidebar ? 60 : 100"
                    :min-size="50"
                >
                    <div class="h-full bg-transparent">
                        <!-- Header -->
                        <div
                            class="p-4 border-b bg-white/60 dark:bg-zinc-900/60 backdrop-blur-sm"
                        >
                            <div class="flex items-center justify-between">
                                <div class="flex items-center gap-4">
                                    <h1
                                        class="text-xl font-bold flex items-center gap-2"
                                    >
                                        <div
                                            class="w-6 h-6 bg-orange-500 rounded flex items-center justify-center"
                                        >
                                            <span
                                                class="text-white text-sm font-bold"
                                                >Y</span
                                            >
                                        </div>
                                        Hacker News
                                    </h1>
                                    <Badge variant="secondary">{{
                                        currentStoryType
                                    }}</Badge>
                                </div>
                                <div class="flex items-center gap-2">
                                    <Button
                                        variant="ghost"
                                        size="sm"
                                        @click="refreshStories"
                                        :disabled="isLoading"
                                        v-tippy="{
                                            content: 'Refresh Stories',
                                            theme: 'translucent',
                                        }"
                                    >
                                        <RefreshCw
                                            :class="[
                                                'h-4 w-4',
                                                isLoading ? 'animate-spin' : '',
                                            ]"
                                        />
                                    </Button>
                                </div>
                            </div>
                        </div>

                        <!-- Stories List -->
                        <ScrollArea
                            class="h-full"
                            :style="{ fontSize: `${fontSize}px` }"
                        >
                            <div class="p-4">
                                <!-- Loading State -->
                                <div
                                    v-if="isLoading && stories.length === 0"
                                    class="space-y-4"
                                >
                                    <div
                                        v-for="n in 10"
                                        :key="n"
                                        class="animate-pulse"
                                    >
                                        <div
                                            class="h-4 bg-gray-300 dark:bg-gray-700 rounded w-3/4 mb-2"
                                        ></div>
                                        <div
                                            class="h-3 bg-gray-200 dark:bg-gray-600 rounded w-1/2"
                                        ></div>
                                    </div>
                                </div>

                                <!-- Error State -->
                                <div v-else-if="error" class="text-center py-8">
                                    <div class="text-red-500 mb-4">
                                        <AlertTriangle
                                            class="h-8 w-8 mx-auto mb-2"
                                        />
                                        <p>Failed to load stories</p>
                                        <p class="text-sm text-gray-500 mt-1">
                                            {{ error }}
                                        </p>
                                    </div>
                                    <Button
                                        @click="refreshStories"
                                        variant="outline"
                                    >
                                        Try Again
                                    </Button>
                                </div>

                                <!-- Stories -->
                                <div v-else class="space-y-3">
                                    <Card
                                        v-for="(story, index) in stories"
                                        :key="story.id"
                                        class="cursor-pointer transition-colors hover:bg-gray-50 dark:hover:bg-gray-800"
                                        :class="{
                                            'border-orange-500 border-2':
                                                selectedStory?.id === story.id,
                                        }"
                                        @click="selectStory(story)"
                                    >
                                        <CardContent class="p-4">
                                            <div class="flex gap-4">
                                                <div
                                                    class="text-gray-500 dark:text-gray-400 text-sm font-mono min-w-[2rem]"
                                                >
                                                    {{ index + 1 }}.
                                                </div>
                                                <div class="flex-1 min-w-0">
                                                    <h3
                                                        class="font-medium leading-tight mb-2 hover:text-orange-600 dark:hover:text-orange-400"
                                                    >
                                                        {{ story.title }}
                                                    </h3>
                                                    <div
                                                        class="flex items-center gap-4 text-sm text-gray-500 dark:text-gray-400"
                                                    >
                                                        <div
                                                            class="flex items-center gap-1"
                                                        >
                                                            <ArrowUp
                                                                class="h-3 w-3"
                                                            />
                                                            {{ story.score }}
                                                        </div>
                                                        <div
                                                            class="flex items-center gap-1"
                                                        >
                                                            <User
                                                                class="h-3 w-3"
                                                            />
                                                            {{ story.by }}
                                                        </div>
                                                        <div
                                                            class="flex items-center gap-1"
                                                        >
                                                            <Clock
                                                                class="h-3 w-3"
                                                            />
                                                            {{
                                                                formatTime(
                                                                    story.time,
                                                                )
                                                            }}
                                                        </div>
                                                        <div
                                                            v-if="
                                                                story.descendants
                                                            "
                                                            class="flex items-center gap-1"
                                                        >
                                                            <MessageCircle
                                                                class="h-3 w-3"
                                                            />
                                                            {{
                                                                story.descendants
                                                            }}
                                                            comments
                                                        </div>
                                                    </div>
                                                    <div
                                                        v-if="story.url"
                                                        class="mt-2"
                                                    >
                                                        <a
                                                            :href="story.url"
                                                            target="_blank"
                                                            class="text-xs text-blue-600 dark:text-blue-400 hover:underline"
                                                            @click.stop
                                                        >
                                                            {{
                                                                getDomain(
                                                                    story.url,
                                                                )
                                                            }}
                                                            <ExternalLink
                                                                class="h-3 w-3 inline"
                                                            />
                                                        </a>
                                                    </div>
                                                </div>
                                            </div>
                                        </CardContent>
                                    </Card>
                                </div>

                                <!-- Load More -->
                                <div
                                    v-if="stories.length > 0 && !isLoading"
                                    class="text-center py-4"
                                >
                                    <Button
                                        @click="loadMoreStories"
                                        variant="outline"
                                    >
                                        Load More Stories
                                    </Button>
                                </div>
                            </div>
                        </ScrollArea>
                    </div>
                </ResizablePanel>

                <!-- Comments Sidebar -->
                <ResizableHandle v-if="showSidebar" />
                <ResizablePanel
                    v-if="showSidebar"
                    :default-size="40"
                    :min-size="25"
                    :max-size="50"
                >
                    <div
                        class="h-full bg-white/40 dark:bg-zinc-900/40 border-l"
                    >
                        <div
                            class="p-4 border-b bg-white/60 dark:bg-zinc-900/60 backdrop-blur-sm"
                        >
                            <h3 class="font-semibold">Comments</h3>
                            <p
                                v-if="selectedStory"
                                class="text-sm text-gray-500 dark:text-gray-400 mt-1"
                            >
                                {{ selectedStory.descendants || 0 }} comments
                            </p>
                        </div>
                        <ScrollArea
                            class="h-full"
                            :style="{ fontSize: `${fontSize}px` }"
                        >
                            <div class="p-4">
                                <div
                                    v-if="!selectedStory"
                                    class="text-center text-gray-500 dark:text-gray-400 py-8"
                                >
                                    Select a story to view comments
                                </div>
                                <div
                                    v-else-if="commentsLoading"
                                    class="space-y-4"
                                >
                                    <div
                                        v-for="n in 5"
                                        :key="n"
                                        class="animate-pulse"
                                    >
                                        <div
                                            class="h-3 bg-gray-300 dark:bg-gray-700 rounded w-full mb-2"
                                        ></div>
                                        <div
                                            class="h-3 bg-gray-200 dark:bg-gray-600 rounded w-3/4"
                                        ></div>
                                    </div>
                                </div>
                                <div
                                    v-else-if="comments.length === 0"
                                    class="text-center text-gray-500 dark:text-gray-400 py-8"
                                >
                                    No comments yet
                                </div>
                                <div v-else class="space-y-4">
                                    <div
                                        v-for="comment in comments"
                                        :key="comment.id"
                                        class="border-l-2 border-gray-200 dark:border-gray-700 pl-3"
                                    >
                                        <div
                                            class="text-xs text-gray-500 dark:text-gray-400 mb-1"
                                        >
                                            {{ comment.by }} •
                                            {{ formatTime(comment.time) }}
                                        </div>
                                        <div
                                            class="text-sm leading-relaxed prose prose-sm dark:prose-invert max-w-none"
                                            v-html="comment.text"
                                        ></div>
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
            class="status-bar border-t bg-white/80 dark:bg-zinc-900/80 backdrop-blur-sm px-4 py-1 text-xs text-gray-500 dark:text-gray-400 flex items-center justify-between shrink-0"
        >
            <div class="flex items-center gap-4">
                <span>{{ stories.length }} stories loaded</span>
                <span v-if="selectedStory">{{ selectedStory.title }}</span>
            </div>
            <div class="flex items-center gap-4">
                <span>Zoom: {{ Math.round((fontSize / 14) * 100) }}%</span>
                <span v-if="isLoading">Loading...</span>
            </div>
        </div>
    </div>
</template>

<script>
import { ref, reactive, computed, onMounted, watch } from "vue";
import useSettings from "@/composables/useSettings";
import { useShortcut } from "@/composables/useShortcut";
import {
    TrendingUp,
    Clock,
    Star,
    RefreshCw,
    Check,
    PanelRight,
    ZoomIn,
    ZoomOut,
    Moon,
    Sun,
    Settings,
    AlertTriangle,
    ArrowUp,
    User,
    MessageCircle,
    ExternalLink,
    Plus,
    Edit,
    Trash2,
} from "lucide-vue-next";

export default {
    name: "HackerNews",
    setup() {
        // Settings
        const { getSetting, setSetting, registerApp } = useSettings();
        const { registerLocalShortcut } = useShortcut();

        // State
        const stories = ref([]);
        const comments = ref([]);
        const selectedStory = ref(null);
        const currentStoryType = ref("Top Stories");
        const isLoading = ref(false);
        const commentsLoading = ref(false);
        const error = ref(null);
        const showSidebar = ref(getSetting("hnShowSidebar", true));
        const isDarkMode = ref(getSetting("hnDarkMode", false));
        const fontSize = ref(getSetting("hnFontSize", 14));
        const currentPage = ref(0);
        const storyIds = ref([]);

        // API Base URL
        const API_BASE = "https://hacker-news.firebaseio.com/v0";

        // Register app
        // registerApp({
        //   name: 'Hacker News',
        //   version: '1.0.0',
        //   description: 'A simple Hacker News client'
        // })

        // API Functions
        const fetchStoryIds = async (type = "top") => {
            const response = await fetch(`${API_BASE}/${type}stories.json`);
            if (!response.ok) throw new Error("Failed to fetch story IDs");
            return response.json();
        };

        const fetchStory = async (id) => {
            const response = await fetch(`${API_BASE}/item/${id}.json`);
            if (!response.ok) throw new Error("Failed to fetch story");
            return response.json();
        };

        const fetchComment = async (id) => {
            const response = await fetch(`${API_BASE}/item/${id}.json`);
            if (!response.ok) throw new Error("Failed to fetch comment");
            return response.json();
        };

        // Story Loading Functions
        const loadStories = async (type = "top", reset = true) => {
            try {
                isLoading.value = true;
                error.value = null;

                if (reset) {
                    stories.value = [];
                    currentPage.value = 0;
                }

                const ids = await fetchStoryIds(type);
                storyIds.value = ids;

                const startIndex = currentPage.value * 30;
                const endIndex = Math.min(startIndex + 30, ids.length);
                const pageIds = ids.slice(startIndex, endIndex);

                const storyPromises = pageIds.map((id) => fetchStory(id));
                const pageStories = await Promise.all(storyPromises);

                const validStories = pageStories.filter(
                    (story) => story && story.title,
                );

                if (reset) {
                    stories.value = validStories;
                } else {
                    stories.value.push(...validStories);
                }

                currentPage.value++;
            } catch (err) {
                error.value = err.message;
                console.error("Failed to load stories:", err);
            } finally {
                isLoading.value = false;
            }
        };

        const loadTopStories = () => {
            currentStoryType.value = "Top Stories";
            loadStories("top");
        };

        const loadNewStories = () => {
            currentStoryType.value = "New Stories";
            loadStories("new");
        };

        const loadBestStories = () => {
            currentStoryType.value = "Best Stories";
            loadStories("best");
        };

        const loadMoreStories = () => {
            const type = currentStoryType.value.toLowerCase().split(" ")[0];
            loadStories(type, false);
        };

        const refreshStories = () => {
            const type = currentStoryType.value.toLowerCase().split(" ")[0];
            loadStories(type);
        };

        // Comments Loading
        const loadComments = async (story) => {
            if (!story.kids || story.kids.length === 0) {
                comments.value = [];
                return;
            }

            try {
                commentsLoading.value = true;
                const commentPromises = story.kids
                    .slice(0, 20)
                    .map((id) => fetchComment(id));
                const commentData = await Promise.all(commentPromises);
                comments.value = commentData.filter(
                    (comment) => comment && comment.text,
                );
            } catch (err) {
                console.error("Failed to load comments:", err);
                comments.value = [];
            } finally {
                commentsLoading.value = false;
            }
        };

        // Story Selection
        const selectStory = (story) => {
            selectedStory.value = story;
            if (showSidebar.value) {
                loadComments(story);
            }
        };

        // UI Controls
        const toggleSidebar = () => {
            showSidebar.value = !showSidebar.value;
            setSetting("hnShowSidebar", showSidebar.value);
            if (showSidebar.value && selectedStory.value) {
                loadComments(selectedStory.value);
            }
        };

        const toggleDarkMode = () => {
            isDarkMode.value = !isDarkMode.value;
            setSetting("hnDarkMode", isDarkMode.value);
        };

        const zoomIn = () => {
            fontSize.value = Math.min(fontSize.value + 2, 24);
            setSetting("hnFontSize", fontSize.value);
        };

        const zoomOut = () => {
            fontSize.value = Math.max(fontSize.value - 2, 10);
            setSetting("hnFontSize", fontSize.value);
        };

        const openSettings = () => {
            // Could open a settings modal in the future
            console.log("Settings clicked");
        };

        // Utility Functions
        const formatTime = (timestamp) => {
            const now = Date.now() / 1000;
            const diff = now - timestamp;
            const hours = Math.floor(diff / 3600);
            const days = Math.floor(hours / 24);

            if (days > 0) return `${days}d ago`;
            if (hours > 0) return `${hours}h ago`;
            return `${Math.floor(diff / 60)}m ago`;
        };

        const getDomain = (url) => {
            try {
                return new URL(url).hostname.replace("www.", "");
            } catch {
                return url;
            }
        };

        // Keyboard Shortcuts
        registerLocalShortcut("ctrl+t", loadTopStories);
        registerLocalShortcut("ctrl+n", loadNewStories);
        registerLocalShortcut("ctrl+b", loadBestStories);
        registerLocalShortcut("f5", refreshStories);
        registerLocalShortcut("ctrl+d", toggleSidebar);
        registerLocalShortcut("ctrl+=", zoomIn);
        registerLocalShortcut("ctrl+-", zoomOut);

        // Watch sidebar changes
        watch(showSidebar, (newValue) => {
            if (newValue && selectedStory.value) {
                loadComments(selectedStory.value);
            }
        });

        // Initialize
        onMounted(() => {
            loadTopStories();
        });

        return {
            // State
            stories,
            comments,
            selectedStory,
            currentStoryType,
            isLoading,
            commentsLoading,
            error,
            showSidebar,
            isDarkMode,
            fontSize,

            // Actions
            loadTopStories,
            loadNewStories,
            loadBestStories,
            loadMoreStories,
            refreshStories,
            selectStory,
            toggleSidebar,
            toggleDarkMode,
            zoomIn,
            zoomOut,
            openSettings,

            // Utilities
            formatTime,
            getDomain,

            // Icons
            TrendingUp,
            Clock,
            Star,
            RefreshCw,
            Check,
            PanelRight,
            ZoomIn,
            ZoomOut,
            Moon,
            Sun,
            Settings,
            AlertTriangle,
            ArrowUp,
            User,
            MessageCircle,
            ExternalLink,
        };
    },
};
</script>

<style scoped>
.status-bar {
    height: 28px;
}

@media (max-width: 768px) {
    .status-bar {
        font-size: 10px;
        padding: 0 8px;
    }
}

@media (max-width: 640px) {
    .status-bar {
        display: none;
    }
    .menubar-trigger {
        font-size: 12px;
        padding: 4px 8px;
    }
}

.prose {
    font-size: inherit;
}

.prose p {
    margin: 0.5rem 0;
}

.prose a {
    color: #3b82f6;
    text-decoration: none;
}

.prose a:hover {
    text-decoration: underline;
}

.dark .prose a {
    color: #60a5fa;
}
</style>
