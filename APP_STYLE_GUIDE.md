# Jun!s App Style Guide

A comprehensive guide for developing applications within the Jun!s Desktop environment using Shadcn-Vue components, focusing on UI/UX consistency and the unique window interaction model.

## Table of Contents

1. [Core Principles](#core-principles)
2. [Window Interaction Model](#window-interaction-model)
3. [Shadcn-Vue Component Usage](#shadcn-vue-component-usage)
4. [App Structure Patterns](#app-structure-patterns)
5. [Layout Guidelines](#layout-guidelines)
6. [Component Guidelines](#component-guidelines)
7. [Code Standards](#code-standards)

## Core Principles

### 1. Transparent Backgrounds
- **NEVER use solid backgrounds** - Always use transparent backgrounds to preserve glassmorphism
- The window manager handles backdrop blur and transparency effects
- Use `bg-transparent` as the base for all app containers

### 2. Pointer Events Model
- **Use `pointer-events-auto` on interactive containers** - the property inherits to children
- **Elements WITHOUT `pointer-events-auto` become part of the window drag area**
- This allows users to drag the window by clicking on non-interactive areas
- Critical for Jun!s Desktop's window management UX

### 3. Blur Usage
- **Only use `backdrop-blur-sm` when content is layered OVER other app content**
- **Never use blur on base panels** - the window already provides blur
- Use blur on: modals, dropdowns, tooltips, overlays
- Don't use blur on: sidebars, toolbars, main content areas

### 4. Shadcn-Vue First
- Use Shadcn-Vue components for all UI elements
- Follow their established patterns and APIs
- Don't recreate existing functionality

### 5. User Interface Dialogs
- **Never use browser native dialogs** - `alert()`, `confirm()`, `prompt()`
- **Always use Shadcn-Vue Dialog components** for user interaction
- Use `Sonner` for notifications and feedback
- Use `Dialog` for confirmations and forms
- Use `AlertDialog` for destructive actions

#### Examples:

```typescript
// ❌ WRONG - Never use browser native dialogs
alert('File saved successfully!')
const result = confirm('Are you sure you want to delete this file?')
const name = prompt('Enter file name:', 'untitled.txt')

// ✅ CORRECT - Use Shadcn-Vue components
// For notifications
import { toast } from 'vue-sonner'
toast.success('File saved successfully!')

// For confirmations
const isDeleteConfirmOpen = ref(false)
const handleDelete = () => {
  isDeleteConfirmOpen.value = true
}

// For input prompts
const isRenameDialogOpen = ref(false)
const newFileName = ref('untitled.txt')
```

```vue
<!-- ✅ CORRECT - Confirmation dialog -->
<AlertDialog v-model:open="isDeleteConfirmOpen">
  <AlertDialogContent class="bg-background/90 backdrop-blur-sm">
    <AlertDialogHeader>
      <AlertDialogTitle>Are you sure?</AlertDialogTitle>
      <AlertDialogDescription>
        This action cannot be undone. This will permanently delete the file.
      </AlertDialogDescription>
    </AlertDialogHeader>
    <AlertDialogFooter>
      <AlertDialogCancel>Cancel</AlertDialogCancel>
      <AlertDialogAction @click="confirmDelete">Delete</AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>

<!-- ✅ CORRECT - Input dialog -->
<Dialog v-model:open="isRenameDialogOpen">
  <DialogContent class="bg-background/90 backdrop-blur-sm">
    <DialogHeader>
      <DialogTitle>Rename File</DialogTitle>
      <DialogDescription>
        Enter a new name for the file
      </DialogDescription>
    </DialogHeader>
    <div class="space-y-4">
      <Input
        v-model="newFileName"
        placeholder="Enter file name"
        @keyup.enter="confirmRename"
      />
    </div>
    <DialogFooter>
      <Button variant="outline" @click="isRenameDialogOpen = false">
        Cancel
      </Button>
      <Button @click="confirmRename">
        Rename
      </Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
```

## Window Interaction Model

### The Pointer Events Pattern

```vue
<template>
  <!-- Base app container - NO pointer-events-auto = draggable area -->
  <div class="h-full w-full bg-transparent">

    <!-- Interactive elements MUST have pointer-events-auto -->
    <Button class="pointer-events-auto" @click="handleClick">
      Click me (not draggable)
    </Button>

    <!-- Non-interactive content becomes drag area -->
    <div class="p-4">
      <h1>This text area can drag the window</h1>
    </div>

    <!-- Input elements need pointer-events-auto -->
    <Input class="pointer-events-auto" v-model="value" />

    <!-- Entire interactive panels need pointer-events-auto -->
    <div class="pointer-events-auto bg-background/20 border rounded-lg p-4">
      <h2>This entire panel is interactive</h2>
      <p>Clicking anywhere here won't drag the window</p>
      <Button @click="action">Button inside interactive panel (inherits pointer-events)</Button>
    </div>
  </div>
</template>
```

### Window Structure

```vue
<template>
  <!-- App root - transparent background, no pointer-events-auto -->
  <div class="h-full w-full bg-transparent">

    <!-- Content areas that should be draggable -->
    <div class="flex h-full">

      <!-- Sidebar - interactive area -->
      <div class="pointer-events-auto w-64 bg-background/20 border-r">
        <ScrollArea class="h-full">
          <!-- Sidebar content -->
        </ScrollArea>
      </div>

      <!-- Main content area -->
      <div class="flex-1 flex flex-col">

        <!-- Toolbar - interactive -->
        <div class="pointer-events-auto flex items-center justify-between p-4 border-b bg-background/20">
          <div class="flex items-center space-x-2">
            <Button size="sm">
              <Plus class="w-4 h-4" />
            </Button>
            <Button size="sm">
              <Folder class="w-4 h-4" />
            </Button>
          </div>

          <Input placeholder="Search..." class="max-w-sm" />
        </div>

        <!-- Content area - mixed draggable/interactive -->
        <div class="flex-1 p-4">
          <!-- Empty spaces are draggable -->
          <div class="mb-4">
            <h1>File Manager</h1>
            <p>This area can drag the window</p>
          </div>

          <!-- File items are interactive -->
          <div class="space-y-2">
            <div class="pointer-events-auto flex items-center space-x-2 p-2 rounded hover:bg-accent/50">
              <File class="w-4 h-4" />
              <span>document.txt</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
```

## Shadcn-Vue Component Usage

### Layout Components

```vue
<template>
  <!-- Use ResizablePanelGroup for split layouts -->
  <ResizablePanelGroup direction="horizontal" class="h-full pointer-events-auto">
    <ResizablePanel :default-size="25" :min-size="20">
      <div class="h-full bg-background/20 border-r">
        <!-- Sidebar content -->
      </div>
    </ResizablePanel>

    <ResizableHandle />

    <ResizablePanel :default-size="75">
      <div class="h-full bg-transparent">
        <!-- Main content -->
      </div>
    </ResizablePanel>
  </ResizablePanelGroup>
</template>
```

### Tab System

```vue
<template>
  <Tabs v-model:value="activeTab" class="h-full pointer-events-auto">
    <TabsList class="bg-background/20">
      <TabsTrigger v-for="tab in tabs" :key="tab.id" :value="tab.id">
        {{ tab.name }}
      </TabsTrigger>
    </TabsList>

    <TabsContent v-for="tab in tabs" :key="tab.id" :value="tab.id" class="h-full">
      <ScrollArea class="h-full">
        <!-- Tab content -->
      </ScrollArea>
    </TabsContent>
  </Tabs>
</template>
```

### Context Menu

```vue
<template>
  <ContextMenu>
    <ContextMenuTrigger class="pointer-events-auto">
      <!-- Right-click target -->
    </ContextMenuTrigger>

    <ContextMenuContent class="bg-background/90 backdrop-blur-sm">
      <ContextMenuItem>
        <Copy class="w-4 h-4 mr-2" />
        Copy
      </ContextMenuItem>
      <ContextMenuItem>
        <Cut class="w-4 h-4 mr-2" />
        Cut
      </ContextMenuItem>
      <ContextMenuSeparator />
      <ContextMenuItem class="text-destructive">
        <Trash class="w-4 h-4 mr-2" />
        Delete
      </ContextMenuItem>
    </ContextMenuContent>
  </ContextMenu>
</template>
```

### Modal Dialogs

```vue
<template>
  <Dialog v-model:open="isOpen">
    <DialogContent class="bg-background/90 backdrop-blur-sm">
      <DialogHeader>
        <DialogTitle>{{ title }}</DialogTitle>
        <DialogDescription>{{ description }}</DialogDescription>
      </DialogHeader>

      <div class="space-y-4">
        <!-- Modal content -->
      </div>

      <DialogFooter>
        <Button variant="outline" @click="handleCancel">
          Cancel
        </Button>
        <Button @click="handleConfirm">
          Confirm
        </Button>
      </DialogFooter>
    </DialogContent>
  </Dialog>
</template>
```

### Form Components

```vue
<template>
  <form @submit.prevent="handleSubmit" class="pointer-events-auto space-y-6">
    <div class="space-y-4">
      <div class="space-y-2">
        <Label for="name">Name</Label>
        <Input
          id="name"
          v-model="form.name"
          placeholder="Enter name"
        />
      </div>

      <div class="space-y-2">
        <Label for="description">Description</Label>
        <Textarea
          id="description"
          v-model="form.description"
          placeholder="Enter description"
        />
      </div>

      <div class="space-y-2">
        <Label for="category">Category</Label>
        <Select v-model="form.category">
          <SelectTrigger>
            <SelectValue placeholder="Select category" />
          </SelectTrigger>
          <SelectContent class="bg-background/90 backdrop-blur-sm">
            <SelectItem value="productivity">Productivity</SelectItem>
            <SelectItem value="media">Media</SelectItem>
            <SelectItem value="development">Development</SelectItem>
          </SelectContent>
        </Select>
      </div>

      <div class="flex items-center space-x-2">
        <Checkbox id="enabled" v-model:checked="form.enabled" />
        <Label for="enabled">Enabled</Label>
      </div>
    </div>

    <div class="flex justify-end space-x-2">
      <Button type="button" variant="outline" @click="handleCancel">
        Cancel
      </Button>
      <Button type="submit">
        Save
      </Button>
    </div>
  </form>
</template>
```

### Data Display

```vue
<template>
  <div class="pointer-events-auto space-y-4">
    <!-- Search and actions -->
    <div class="flex items-center justify-between">
      <Input
        v-model="searchQuery"
        placeholder="Search..."
        class="max-w-sm"
      />

      <div class="flex items-center space-x-2">
        <Button variant="outline" size="sm">
          <RefreshCw class="w-4 h-4" />
        </Button>
        <Button size="sm">
          <Plus class="w-4 h-4" />
        </Button>
      </div>
    </div>

    <!-- Table -->
    <div class="border rounded-lg bg-background/20">
      <Table>
        <TableHeader>
          <TableRow>
            <TableHead>Name</TableHead>
            <TableHead>Size</TableHead>
            <TableHead>Modified</TableHead>
            <TableHead class="text-right">Actions</TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          <TableRow v-for="item in filteredItems" :key="item.id">
            <TableCell>{{ item.name }}</TableCell>
            <TableCell>{{ item.size }}</TableCell>
            <TableCell>{{ item.modified }}</TableCell>
            <TableCell class="text-right">
              <DropdownMenu>
                <DropdownMenuTrigger asChild>
                  <Button variant="ghost" size="sm">
                    <MoreHorizontal class="w-4 h-4" />
                  </Button>
                </DropdownMenuTrigger>
                <DropdownMenuContent class="bg-background/90 backdrop-blur-sm">
                  <DropdownMenuItem @click="handleEdit(item)">
                    Edit
                  </DropdownMenuItem>
                  <DropdownMenuItem @click="handleDelete(item)" class="text-destructive">
                    Delete
                  </DropdownMenuItem>
                </DropdownMenuContent>
              </DropdownMenu>
            </TableCell>
          </TableRow>
        </TableBody>
      </Table>
    </div>
  </div>
</template>
```

## App Structure Patterns

### File Manager Pattern

```vue
<template>
  <div class="h-full w-full bg-transparent">
    <ResizablePanelGroup direction="horizontal" class="h-full pointer-events-auto">
      <!-- File tree sidebar -->
      <ResizablePanel :default-size="25" :min-size="20">
        <div class="h-full bg-background/20 border-r">
          <div class="p-4 border-b">
            <h2 class="font-semibold">Explorer</h2>
          </div>

          <ScrollArea class="h-full">
            <div class="p-2 space-y-1">
              <Collapsible v-for="folder in folders" :key="folder.id">
                <CollapsibleTrigger class="flex items-center w-full p-2 hover:bg-accent/50 rounded">
                  <ChevronRight class="w-4 h-4 mr-2" />
                  <Folder class="w-4 h-4 mr-2" />
                  {{ folder.name }}
                </CollapsibleTrigger>
                <CollapsibleContent class="pl-6">
                  <!-- Nested items -->
                </CollapsibleContent>
              </Collapsible>
            </div>
          </ScrollArea>
        </div>
      </ResizablePanel>

      <ResizableHandle />

      <!-- Main content -->
      <ResizablePanel :default-size="75">
        <div class="h-full flex flex-col bg-transparent">
          <!-- Toolbar -->
          <div class="pointer-events-auto flex items-center justify-between p-4 border-b bg-background/20">
            <Breadcrumb>
              <BreadcrumbList>
                <BreadcrumbItem v-for="(item, index) in breadcrumbItems" :key="index">
                  <BreadcrumbLink
                    v-if="index < breadcrumbItems.length - 1"
                    @click="navigateTo(item.path)"
                  >
                    {{ item.name }}
                  </BreadcrumbLink>
                  <BreadcrumbPage v-else>
                    {{ item.name }}
                  </BreadcrumbPage>
                </BreadcrumbItem>
                <BreadcrumbSeparator v-if="index < breadcrumbItems.length - 1" />
              </BreadcrumbList>
            </Breadcrumb>

            <div class="flex items-center space-x-2">
              <Button variant="outline" size="sm">
                <ArrowLeft class="w-4 h-4" />
              </Button>
              <Button variant="outline" size="sm">
                <RefreshCw class="w-4 h-4" />
              </Button>
            </div>
          </div>

          <!-- File grid -->
          <ScrollArea class="flex-1">
            <div class="p-4">
              <!-- Empty area for window dragging -->
              <div class="mb-4">
                <h1 class="text-2xl font-semibold">Files</h1>
                <p class="text-muted-foreground">This area can drag the window</p>
              </div>

              <!-- File items -->
              <div class="grid grid-cols-4 gap-4">
                <ContextMenu v-for="file in files" :key="file.id">
                  <ContextMenuTrigger>
                    <div class="pointer-events-auto flex flex-col items-center p-4 rounded hover:bg-accent/50">
                      <FileIcon class="w-8 h-8 mb-2" />
                      <span class="text-sm text-center">{{ file.name }}</span>
                    </div>
                  </ContextMenuTrigger>
                  <ContextMenuContent class="bg-background/90 backdrop-blur-sm">
                    <ContextMenuItem>Open</ContextMenuItem>
                    <ContextMenuItem>Rename</ContextMenuItem>
                    <ContextMenuSeparator />
                    <ContextMenuItem class="text-destructive">Delete</ContextMenuItem>
                  </ContextMenuContent>
                </ContextMenu>
              </div>
            </div>
          </ScrollArea>
        </div>
      </ResizablePanel>
    </ResizablePanelGroup>
  </div>
</template>
```

### Code Editor Pattern

```vue
<template>
  <div class="h-full w-full bg-transparent">
    <ResizablePanelGroup direction="horizontal" class="h-full pointer-events-auto">
      <!-- File explorer -->
      <ResizablePanel :default-size="20" :min-size="15">
        <div class="h-full bg-background/20 border-r">
          <div class="p-4 border-b">
            <h2 class="font-semibold">Files</h2>
          </div>
          <ScrollArea class="h-full">
            <!-- File tree -->
          </ScrollArea>
        </div>
      </ResizablePanel>

      <ResizableHandle />

      <!-- Editor area -->
      <ResizablePanel :default-size="60">
        <div class="h-full flex flex-col bg-transparent">
          <!-- Tabs -->
          <Tabs v-model:value="activeTab" class="h-full">
            <TabsList class="justify-start bg-background/20">
              <TabsTrigger v-for="tab in openTabs" :key="tab.id" :value="tab.id">
                <div class="flex items-center">
                  <span>{{ tab.name }}</span>
                  <Button
                    variant="ghost"
                    size="sm"
                    class="ml-2 h-4 w-4 p-0"
                    @click.stop="closeTab(tab.id)"
                  >
                    <X class="w-3 h-3" />
                  </Button>
                </div>
              </TabsTrigger>
            </TabsList>

            <TabsContent v-for="tab in openTabs" :key="tab.id" :value="tab.id" class="flex-1">
              <div class="h-full bg-background/10">
                <!-- Editor component would go here -->
                <textarea
                  v-model="tab.content"
                  class="w-full h-full p-4 bg-transparent border-0 outline-0 resize-none font-mono"
                  placeholder="Start typing..."
                />
              </div>
            </TabsContent>
          </Tabs>
        </div>
      </ResizablePanel>

      <ResizableHandle />

      <!-- Right sidebar -->
      <ResizablePanel :default-size="20" :min-size="15">
        <div class="h-full bg-background/20 border-l">
          <Tabs default-value="outline" class="h-full">
            <TabsList class="grid w-full grid-cols-2">
              <TabsTrigger value="outline">Outline</TabsTrigger>
              <TabsTrigger value="problems">Problems</TabsTrigger>
            </TabsList>

            <TabsContent value="outline" class="h-full">
              <ScrollArea class="h-full">
                <div class="p-4">
                  <h3 class="font-semibold mb-2">Document Outline</h3>
                  <!-- Outline content -->
                </div>
              </ScrollArea>
            </TabsContent>

            <TabsContent value="problems" class="h-full">
              <ScrollArea class="h-full">
                <div class="p-4">
                  <h3 class="font-semibold mb-2">Problems</h3>
                  <!-- Problems content -->
                </div>
              </ScrollArea>
            </TabsContent>
          </Tabs>
        </div>
      </ResizablePanel>
    </ResizablePanelGroup>
  </div>
</template>
```

### Settings Pattern

```vue
<template>
  <div class="h-full w-full bg-transparent">
    <ResizablePanelGroup direction="horizontal" class="h-full pointer-events-auto">
      <!-- Settings categories -->
      <ResizablePanel :default-size="25" :min-size="20">
        <div class="h-full bg-background/20 border-r">
          <div class="p-4 border-b">
            <h2 class="font-semibold">Settings</h2>
          </div>

          <ScrollArea class="h-full">
            <nav class="space-y-1 p-2">
              <Button
                v-for="category in categories"
                :key="category.id"
                variant="ghost"
                class="w-full justify-start"
                :class="{ 'bg-accent': activeCategory === category.id }"
                @click="setActiveCategory(category.id)"
              >
                <component :is="category.icon" class="w-4 h-4 mr-2" />
                {{ category.name }}
              </Button>
            </nav>
          </ScrollArea>
        </div>
      </ResizablePanel>

      <ResizableHandle />

      <!-- Settings content -->
      <ResizablePanel :default-size="75">
        <ScrollArea class="h-full">
          <div class="p-6 space-y-8">
            <!-- Settings sections -->
            <div v-for="section in currentSections" :key="section.id" class="space-y-4">
              <div>
                <h3 class="text-lg font-semibold">{{ section.title }}</h3>
                <p class="text-sm text-muted-foreground">{{ section.description }}</p>
              </div>

              <div class="space-y-4">
                <div v-for="setting in section.settings" :key="setting.key" class="flex items-center justify-between">
                  <div class="space-y-1">
                    <Label>{{ setting.label }}</Label>
                    <p class="text-sm text-muted-foreground">{{ setting.description }}</p>
                  </div>

                  <div class="flex items-center space-x-2">
                    <Switch
                      v-if="setting.type === 'boolean'"
                      v-model:checked="settings[setting.key]"
                    />

                    <Select
                      v-else-if="setting.type === 'select'"
                      v-model="settings[setting.key]"
                    >
                      <SelectTrigger class="w-48">
                        <SelectValue />
                      </SelectTrigger>
                      <SelectContent class="bg-background/90 backdrop-blur-sm">
                        <SelectItem
                          v-for="option in setting.options"
                          :key="option.value"
                          :value="option.value"
                        >
                          {{ option.label }}
                        </SelectItem>
                      </SelectContent>
                    </Select>

                    <Input
                      v-else
                      v-model="settings[setting.key]"
                      class="w-48"
                    />
                  </div>
                </div>
              </div>
            </div>
          </div>
        </ScrollArea>
      </ResizablePanel>
    </ResizablePanelGroup>
  </div>
</template>
```

## Layout Guidelines

### Responsive Design

```vue
<template>
  <!-- Mobile: Use Sheet for sidebars -->
  <div class="md:hidden">
    <Sheet v-model:open="sidebarOpen">
      <SheetTrigger asChild>
        <Button variant="outline" size="sm" class="pointer-events-auto">
          <Menu class="w-4 h-4" />
        </Button>
      </SheetTrigger>
      <SheetContent side="left" class="w-64 bg-background/90 backdrop-blur-sm">
        <SheetHeader>
          <SheetTitle>Navigation</SheetTitle>
        </SheetHeader>
        <!-- Mobile sidebar content -->
      </SheetContent>
    </Sheet>
  </div>

  <!-- Desktop: Use ResizablePanelGroup -->
  <div class="hidden md:block h-full">
    <ResizablePanelGroup direction="horizontal" class="h-full pointer-events-auto">
      <!-- Desktop layout -->
    </ResizablePanelGroup>
  </div>
</template>
```

### Background Patterns

```vue
<template>
  <!-- App container - transparent -->
  <div class="h-full w-full bg-transparent">

    <!-- Interactive panels - subtle background -->
    <div class="pointer-events-auto bg-background/20 border rounded-lg">
      <!-- Content -->
    </div>

    <!-- Modals - more opaque background -->
    <DialogContent class="bg-background/90 backdrop-blur-sm">
      <!-- Modal content -->
    </DialogContent>

    <!-- Dropdowns and menus - high opacity with blur -->
    <DropdownMenuContent class="bg-background/90 backdrop-blur-sm">
      <!-- Menu items -->
    </DropdownMenuContent>
  </div>
</template>
```

## Component Guidelines

### Interactive Elements Checklist

✅ **Add `pointer-events-auto` to containers that need interactivity:**
- Interactive panels and cards
- Form containers
- Sidebar containers
- Tab containers
- Table containers
- Toolbar containers

✅ **Individual components only need `pointer-events-auto` when:**
- Their parent container does NOT have `pointer-events-auto`
- They are isolated interactive elements in a draggable area

❌ **Never add `pointer-events-auto` to:**
- App root containers
- Empty space divs
- Text-only headings and paragraphs
- Individual components inside containers that already have `pointer-events-auto`

### Component Composition

```vue
<template>
  <!-- Correct: Interactive card - children inherit pointer-events -->
  <Card class="pointer-events-auto">
    <CardHeader>
      <CardTitle>{{ title }}</CardTitle>
      <CardDescription>{{ description }}</CardDescription>
    </CardHeader>
    <CardContent>
      <p>All content here is interactive</p>
      <Button @click="handleAction">Action (inherits from parent)</Button>
    </CardContent>
  </Card>

  <!-- Correct: Individual interactive elements in draggable area -->
  <Card>
    <CardContent>
      <p>This text area can drag the window</p>
      <Button class="pointer-events-auto" @click="handleAction">
        Only this button is interactive
      </Button>
    </CardContent>
  </Card>
</template>
```

## Code Standards

### App Registration

```typescript
// Standard app props
interface AppProps {
  width: number
  height: number
  x: number
  y: number
  minimized: boolean
  maximized: boolean
  windowId: string
  darkMode?: boolean
}

// App registration
const { getSetting, setSetting, registerApp } = useSettings()
const { registerLocalShortcut } = useShortcuts()

registerApp({
  name: 'MyApp',
  icon: 'app-icon',
  category: 'productivity'
})
```

### Keyboard Shortcuts

```typescript
const registerAppShortcuts = () => {
  registerLocalShortcut('CmdOrCtrl+N', () => createNew())
  registerLocalShortcut('CmdOrCtrl+O', () => openFile())
  registerLocalShortcut('CmdOrCtrl+S', () => saveFile())
  registerLocalShortcut('CmdOrCtrl+W', () => closeTab())
  registerLocalShortcut('CmdOrCtrl+T', () => newTab())
  registerLocalShortcut('CmdOrCtrl+F', () => showSearch())
}
```

### Error Handling

```typescript
import { toast } from 'vue-sonner'

const handleOperation = async () => {
  try {
    await performOperation()
    toast.success("Operation completed successfully")
  } catch (error) {
    toast.error(error.message)
  }
}
```

## Best Practices

1. **Use `pointer-events-auto` on interactive containers** - the property inherits to children
2. **Keep backgrounds transparent** - Preserve the glassmorphism effect
3. **Use proper Shadcn-Vue components** - Don't reinvent existing functionality
4. **Never use native browser dialogs** - Use `Dialog`, `AlertDialog`, and `Sonner` instead of `alert()`, `confirm()`, `prompt()`
5. **Implement responsive design** - Use `Sheet` for mobile, `ResizablePanelGroup` for desktop
6. **Handle loading states** - Use `Skeleton` components and proper loading indicators
7. **Provide keyboard shortcuts** - Essential for power users
8. **Use proper semantic HTML** - Shadcn-Vue components include proper accessibility
9. **Test window dragging** - Ensure non-interactive areas can drag the window
10. **Handle errors gracefully** - Use `Sonner` for user feedback
11. **Follow consistent patterns** - Use the established app structure patterns

This style guide ensures all Jun!s applications maintain consistent behavior with the unique window interaction model while using proper Shadcn-Vue components and preserving the glassmorphism aesthetic.
