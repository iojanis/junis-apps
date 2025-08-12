# Vue App Development Guide

## Overview

This guide covers how to develop Vue.js applications within the Junis Desktop environment. The system provides a rich set of composables, UI components, and utilities that integrate seamlessly with the desktop application framework.

**App backgrounds should always be transparent to ensure a seamless integration with the window manager.**
**Sidebar background should be transparent with a difference in opacity.**
**All elements that do not have or inherit "pointer-events-auto" will not be clickable and be part of the draggable window.**

## Available Resources

### Core Frameworks & Libraries

#### Vue.js
```javascript
import { ref, reactive, computed, watch, onMounted } from 'vue'
import { defineComponent, h } from 'vue'
```

#### VueUse
```javascript
import { useLocalStorage, useMouse, useWindowSize, useClipboard } from '@vueuse/core'
```

#### Lucide Icons
```javascript
import { Plus, Settings, File, Folder, Save, X } from 'lucide-vue-next'
```

### Desktop Integration Composables

The system provides several desktop-specific composables:

```javascript
// File system operations
import { useFileSystem } from '@/composables/useFileSystem'

// Window management
import { useWindow } from '@/composables/useWindow'

// Application settings (reactive and non-reactive options)
import { useSettings } from '@/composables/useSettings'

// System clipboard
import { useClipboard } from '@/composables/useClipboard'

// System tray integration
import { useTray } from '@/composables/useTray'

// Keyboard shortcuts
import { useShortcut } from '@/composables/useShortcut'
```

### GUI Components

Pre-built GUI components are available for common desktop operations:

```javascript
// Modal components
import ChooseAppModal from '@/components/gui/ChooseAppModal'
import ConfirmPromptModal from '@/components/gui/ConfirmPromptModal'

// File system dialogs
import OpenFileModal from '@/components/gui/OpenFileModal'
import OpenFolderModal from '@/components/gui/OpenFolderModal'
import SaveFileModal from '@/components/gui/SaveFileModal'
import FileSystemDialogBase from '@/components/gui/FileSystemDialogBase'

// Code editor
import CodeEditor from '@/components/gui/CodeEditor'
```

### UI Components (shadcn/ui)

**Important**: shadcn/ui components are globally available and should NOT be imported:

```javascript
// ❌ DON'T DO THIS
import { Button } from '@/components/ui/button'

// ✅ DO THIS - Use directly in templates
<template>
  <Button @click="handleClick">Click me</Button>
  <Card>
    <CardHeader>
      <CardTitle>Title</CardTitle>
    </CardHeader>
    <CardContent>
      Content here
    </CardContent>
  </Card>
</template>
```

Available shadcn/ui components include:
- `Button`, `Input`, `Textarea`, `Select`
- `Card`, `CardHeader`, `CardTitle`, `CardContent`
- `Dialog`, `DialogContent`, `DialogHeader`, `DialogTitle`
- `Badge`, `Alert`, `Progress`, `Separator`
- `Table`, `TableHeader`, `TableRow`, `TableCell`
- And many more categories:

Sidebar
Accordion
Alert
Alert Dialog
Aspect Ratio
Avatar
Badge
Breadcrumb
Button
Calendar
Card
Carousel
Checkbox
Collapsible
Combobox
Command
Context Menu
Data Table
Date Picker
Dialog
Drawer
Dropdown Menu
Form
Hover Card
Input
Label
Menubar
Navigation Menu
Number Field
Pagination
PIN Input
Popover
Progress
Radio Group
Range Calendar
Resizable
Scroll Area
Select
Separator
Sheet
Skeleton
Slider
Sonner
Stepper
Switch
Table
Tabs
Tags Input
Textarea
Toast
Toggle
Toggle Group
Tooltip

## File Structure & Organization

### Virtual File System (VFS)

Your Vue apps can use the Virtual File System for organizing code:

```
/
├── Components/
│   ├── MyComponent.vue
│   ├── Shared/
│   │   └── Header.vue
│   └── Modals/
│       └── CustomModal.vue
├── Utils/
│   ├── helpers.js
│   ├── api.js
│   └── constants.js
└── Composables/
    ├── useCustomLogic.js
    └── useDataFetching.js
```

### Import Patterns

#### VFS Imports
```javascript
// Components
import MyComponent from '@/Components/MyComponent.vue'
import Header from '@/Components/Shared/Header.vue'

// JavaScript utilities
import { helper1, helper2 } from '@/Utils/helpers.js'
import * as api from '@/Utils/api.js'

// Custom composables
import { useCustomLogic } from '@/Composables/useCustomLogic.js'
```

#### Dynamic Imports
```javascript
// Lazy loading components
const MyComponent = defineAsyncComponent(() => import('@/Components/MyComponent.vue'))

// Dynamic module loading
const loadUtility = async () => {
  const { utility } = await import('@/Utils/helpers.js')
  return utility
}
```

## Component Development

### Basic Component Structure

```vue
<template>
  <div class="my-component">
    <h1>{{ title }}</h1>
    <Button @click="handleClick">{{ buttonText }}</Button>
  </div>
</template>

<script>
import { ref, computed } from 'vue'
import { Plus } from 'lucide-vue-next'

export default {
  name: 'MyComponent',
  props: {
    title: {
      type: String,
      default: 'Default Title'
    }
  },
  setup(props) {
    const count = ref(0)

    const buttonText = computed(() =>
      `Clicked ${count.value} times`
    )

    const handleClick = () => {
      count.value++
    }

    return {
      buttonText,
      handleClick,
      Plus
    }
  }
}
</script>

<style scoped>
.my-component {
  padding: 1rem;
}
</style>
```

### Composition API with Desktop Integration

```vue
<script>
import { ref, onMounted } from 'vue'
import { useFileSystem } from '@/composables/useFileSystem'
import { useWindow } from '@/composables/useWindow'
import { useSettings } from '@/composables/useSettings'

export default {
  setup() {
    const { fs } = useFileSystem()
    const { window } = useWindow()
    const { settings } = useSettings()

    const files = ref([])
    const isLoading = ref(false)

    const loadFiles = async () => {
      isLoading.value = true
      try {
        const fileList = await fs.readdir('/some/path')
        files.value = fileList
      } catch (error) {
        console.error('Failed to load files:', error)
      } finally {
        isLoading.value = false
      }
    }

    const openFile = async () => {
      const filePath = await fs.openFile({
        filters: [{ name: 'Text files', extensions: ['txt'] }]
      })
      if (filePath) {
        const content = await fs.readFile(filePath)
        // Process file content
      }
    }

    onMounted(loadFiles)

    return {
      files,
      isLoading,
      loadFiles,
      openFile
    }
  }
}
</script>
```

## Best Practices

### 1. Component Organization

```javascript
// Group related imports
import { ref, reactive, computed, watch } from 'vue'
import { useFileSystem, useWindow } from '@/composables/useFileSystem'
import { Save, X } from 'lucide-vue-next'

// Custom components and utilities
import CustomComponent from '@/Components/CustomComponent.vue'
import { helper } from '@/Utils/helpers.js'
```

### 2. Error Handling

```javascript
setup() {
  const { fs } = useFileSystem()
  const error = ref(null)
  const isLoading = ref(false)

  const safeOperation = async () => {
    try {
      isLoading.value = true
      error.value = null

      const result = await fs.someOperation()
      return result
    } catch (err) {
      error.value = err.message
      console.error('Operation failed:', err)
    } finally {
      isLoading.value = false
    }
  }

  return { error, isLoading, safeOperation }
}
```

### 3. Reactive State Management

```javascript
setup() {
  // Simple reactive state
  const state = reactive({
    user: null,
    preferences: {},
    activeTab: 'home'
  })

  // Computed properties
  const isLoggedIn = computed(() => !!state.user)
  const userName = computed(() => state.user?.name || 'Guest')

  // Watchers
  watch(() => state.activeTab, (newTab) => {
    console.log(`Switched to ${newTab}`)
  })

  return { state, isLoggedIn, userName }
}
```

### 4. File System Integration

```javascript
setup() {
  const { fs } = useFileSystem()

  // File operations
  const saveData = async (data) => {
    const path = await fs.saveFile({
      defaultPath: 'data.json',
      filters: [{ name: 'JSON', extensions: ['json'] }]
    })

    if (path) {
      await fs.writeFile(path, JSON.stringify(data, null, 2))
    }
  }

  const loadData = async () => {
    const path = await fs.openFile({
      filters: [{ name: 'JSON', extensions: ['json'] }]
    })

    if (path) {
      const content = await fs.readFile(path)
      return JSON.parse(content)
    }
  }

  return { saveData, loadData }
}
```

## Common Patterns

### 1. Modal Management

```vue
<template>
  <div>
    <Button @click="showModal = true">Open Modal</Button>

    <ConfirmPromptModal
      v-if="showModal"
      title="Confirm Action"
      message="Are you sure you want to proceed?"
      @confirm="handleConfirm"
      @cancel="showModal = false"
    />
  </div>
</template>

<script>
import { ref } from 'vue'
import ConfirmPromptModal from '@/components/gui/ConfirmPromptModal'

export default {
  components: {
    ConfirmPromptModal
  },
  setup() {
    const showModal = ref(false)

    const handleConfirm = () => {
      console.log('Confirmed!')
      showModal.value = false
    }

    return { showModal, handleConfirm }
  }
}
</script>
```

### 2. Settings Integration

```javascript
setup() {
  const { settings, getSetting, setSetting } = useSettings()

  // Reactive approach - use when you need UI to update automatically
  const theme = computed(() => settings.value.theme || 'light')
  const fontSize = computed(() => settings.value.fontSize || 14)

  // Update settings reactively
  const updateTheme = (newTheme) => {
    settings.value.theme = newTheme
  }

  // Non-reactive approach - use for one-time reads/writes or performance
  const loadInitialSettings = () => {
    const savedTheme = getSetting('theme', 'light')
    const savedFontSize = getSetting('fontSize', 14)
    // Use values without reactivity - better performance for static reads
  }

  const saveUserPreference = (key, value) => {
    setSetting(key, value)
    // Setting is saved but won't trigger Vue reactivity
  }

  return { theme, fontSize, updateTheme, loadInitialSettings, saveUserPreference }
}
```

**When to use each approach:**
- **Reactive (`settings.value`)**: When UI components need to update automatically when settings change
- **Non-reactive (`getSetting`/`setSetting`)**: For one-time reads, initialization, or when you don't need UI updates

### 3. Keyboard Shortcuts

```javascript
setup() {
  const { useShortcut } = useShortcut()

  // Register shortcuts
  useShortcut('ctrl+s', () => {
    console.log('Save shortcut pressed')
  })

  useShortcut('ctrl+o', () => {
    console.log('Open shortcut pressed')
  })
}
```

## Performance Considerations

### 1. Component Lazy Loading

```javascript
// Lazy load heavy components
const HeavyComponent = defineAsyncComponent(() =>
  import('@/Components/HeavyComponent.vue')
)

// With loading and error states
const HeavyComponent = defineAsyncComponent({
  loader: () => import('@/Components/HeavyComponent.vue'),
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorComponent,
  delay: 200,
  timeout: 3000
})
```

### 2. Computed Properties

```javascript
// Expensive computations
const expensiveValue = computed(() => {
  // Cache expensive calculations
  return heavyCalculation(data.value)
})

// Avoid in templates
const formattedData = computed(() => {
  return data.value.map(item => ({
    ...item,
    formatted: formatItem(item)
  }))
})
```

### 3. Watchers

```javascript
// Use immediate: false for heavy operations
watch(searchQuery, async (newQuery) => {
  if (newQuery.length > 2) {
    await searchData(newQuery)
  }
}, { immediate: false, debounce: 300 })
```

## Testing Guidelines

### 1. Component Testing

```javascript
// Test setup functions
const setupComponent = (props = {}) => {
  const wrapper = mount(MyComponent, {
    props,
    global: {
      stubs: ['Button', 'Card'] // Stub shadcn components
    }
  })
  return wrapper
}

// Test composables
const testComposable = () => {
  const { result } = renderHook(() => useCustomLogic())
  return result
}
```

## Common Pitfalls

### 1. Import Issues

```javascript
// ❌ Don't import shadcn components
import { Button } from '@/components/ui/button'

// ❌ Don't use absolute paths
import MyComponent from '/Components/MyComponent.vue'

// ✅ Use proper VFS imports
import MyComponent from '@/Components/MyComponent.vue'
```

### 2. Reactive Issues

```javascript
// ❌ Don't mutate props directly
props.items.push(newItem)

// ✅ Emit events or use local state
const emit = defineEmits(['update:items'])
emit('update:items', [...props.items, newItem])
```

### 3. Memory Leaks

```javascript
// ❌ Don't forget to cleanup
onMounted(() => {
  const interval = setInterval(() => {}, 1000)
  // Missing cleanup
})

// ✅ Always cleanup
onMounted(() => {
  const interval = setInterval(() => {}, 1000)

  onUnmounted(() => {
    clearInterval(interval)
  })
})
```

## Advanced Features

### 1. Custom Composables

```javascript
// @/Composables/useCustomLogic.js
import { ref, computed } from 'vue'
import { useFileSystem } from '@/composables/useFileSystem'

export function useCustomLogic() {
  const { fs } = useFileSystem()
  const data = ref([])

  const processedData = computed(() => {
    return data.value.map(item => ({
      ...item,
      processed: true
    }))
  })

  const loadData = async () => {
    // Custom logic here
  }

  return {
    data,
    processedData,
    loadData
  }
}
```

This guide provides a comprehensive foundation for developing Vue.js applications within the Junis Desktop environment. Focus on leveraging the provided composables and components while following Vue.js best practices.
