<template>
  <div role="tabpanel">
    <!-- Table of Contents -->
    <h1>{{ titleOrTocLabel }}</h1>
    
    <!-- Toolbar -->
    <div class="toc-toolbar">
      <!-- Auto-scroll toggle -->
      <button
        v-bind:class="['toc-toolbar-btn', autoScrollEnabled ? 'active' : '']"
        v-on:click="toggleAutoScroll"
        :title="trans('Toggle auto-scroll to current section')"
      >
        <span class="btn-icon">{{ autoScrollEnabled ? '📍' : '◌' }}</span>
      </button>
      
      <!-- Collapse/Expand All -->
      <button
        class="toc-toolbar-btn"
        v-on:click="toggleCollapseAll"
        :title="trans('Collapse/Expand all sections')"
      >
        <span class="btn-icon">{{ allCollapsed ? '📂' : '📁' }}</span>
      </button>
      
      <!-- Search -->
      <input
        v-model="searchQuery"
        type="text"
        class="toc-search"
        :placeholder="trans('Search...')"
      />
    </div>
    
    <!-- Show the ToC entries -->
    <div
      v-for="(entry, idx) of filteredTableOfContents"
      v-bind:key="idx"
      v-bind:data-line="entry.line"
      v-bind:class="'toc-entry-container toc-heading-' + entry.level"
      draggable="true"
      v-on:click="handleEntryClick(entry.line)"
      v-on:dragstart="startDragging"
      v-on:dragover="dragOver"
      v-on:drop="drop"
    >
      <!-- Collapse toggle area: always render to maintain alignment, but only show button for entries with children -->
      <div class="toc-collapse-area">
        <div
          v-if="hasChildren(entry)"
          v-bind:class="['toc-collapse-toggle', isCollapsed(entry.line) ? 'collapsed' : 'expanded']"
          v-on:click.stop="toggleCollapse(entry.line)"
        >
          <span class="toggle-icon"></span>
        </div>
      </div>
      <div class="toc-level">
        {{ entry.renderedLevel }}
      </div>
      <div
        v-bind:class="{ 'toc-entry': true, 'toc-entry-active': tocEntryIsActive(entry.line) }"
        v-bind:data-line="entry.line"
        v-html="getTocEntryHTML(entry.line)"
      ></div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { trans } from '@common/i18n-renderer'
import { ref, computed, watch, toRef, onMounted } from 'vue'
import sanitizeHtml from 'sanitize-html'
import { CITEPROC_MAIN_DB } from '@dts/common/citeproc'
import { type AnyDescriptor } from '@dts/common/fsal'
import { md2html } from '@common/modules/markdown-utils'
import { useConfigStore, useDocumentTreeStore, useWindowStateStore } from 'source/pinia'

const ipcRenderer = window.ipc
const windowStateStore = useWindowStateStore()
const documentTreeStore = useDocumentTreeStore()
const configStore = useConfigStore()

const emit = defineEmits<{
  (e: 'move-section', data: { from: number, to: number }): void
  (e: 'jump-to-line', line: number): void
}>()

const activeFileDescriptor = ref<AnyDescriptor|null>(null)
const library = ref<string>(CITEPROC_MAIN_DB)

const tableOfContents = computed(() => windowStateStore.tableOfContents)
const tocEntryHTML = ref<string[]>([])

// Collapse state: stores line numbers of collapsed headings
const collapsedHeadings = ref<Set<number>>(new Set())

// Search state
const searchQuery = ref<string>('')

// Auto-scroll state
const autoScrollEnabled = ref<boolean>(true)

// Track the last "collapse all" action state (independent of manual collapses)
// false = last action was "expand all", true = last action was "collapse all"
const lastCollapseAllState = ref<boolean>(false)

watch(toRef(tableOfContents), updateToCHTML)
onMounted(() => {
  updateToCHTML()
  // Initialize auto-scroll from config if available
  autoScrollEnabled.value = configStore.config?.editor?.autoScrollToc ?? true
})

/**
 * Check if a heading has child headings (any following entry with deeper level)
 */
function hasChildren(entry: any): boolean {
  if (!tableOfContents.value) return false
  const currentIdx = tableOfContents.value.findIndex(e => e.line === entry.line)
  if (currentIdx === -1 || currentIdx === tableOfContents.value.length - 1) return false
  
  // Look through all following entries to find any with deeper level
  for (let i = currentIdx + 1; i < tableOfContents.value.length; i++) {
    const nextEntry = tableOfContents.value[i]
    // If we find a sibling or parent level, stop searching
    if (nextEntry.level <= entry.level) {
      break
    }
    // If we find a child (deeper level), return true
    if (nextEntry.level > entry.level) {
      return true
    }
  }
  
  return false
}

/**
 * Check if a heading is currently collapsed
 */
function isCollapsed(line: number): boolean {
  return collapsedHeadings.value.has(line)
}

/**
 * Toggle collapse state for a heading
 */
function toggleCollapse(line: number): void {
  if (collapsedHeadings.value.has(line)) {
    collapsedHeadings.value.delete(line)
  } else {
    collapsedHeadings.value.add(line)
  }
  // Trigger reactivity
  collapsedHeadings.value = new Set(collapsedHeadings.value)
}

/**
 * Toggle auto-scroll feature
 */
function toggleAutoScroll(): void {
  autoScrollEnabled.value = !autoScrollEnabled.value
  // Save to config
  if (configStore.config?.editor) {
    configStore.config.editor.autoScrollToc = autoScrollEnabled.value
    configStore.saveConfig()
  }
}

/**
 * Collapse or expand all sections
 * This action is INDEPENDENT of manual collapse/expand operations
 */
function toggleCollapseAll(): void {
  if (!tableOfContents.value) return
  
  const collapsibleEntries = tableOfContents.value.filter(e => hasChildren(e))
  if (collapsibleEntries.length === 0) return
  
  // Toggle the independent state
  lastCollapseAllState.value = !lastCollapseAllState.value
  
  const newCollapsed = new Set<number>()
  
  if (lastCollapseAllState.value) {
    // Collapse all entries that have children
    for (const entry of collapsibleEntries) {
      newCollapsed.add(entry.line)
    }
  }
  // else: leave newCollapsed empty (expand all)
  
  collapsedHeadings.value = newCollapsed
}

/**
 * Handle click on ToC entry
 */
function handleEntryClick(line: number): void {
  emit('jump-to-line', line)
}

/**
 * Get the HTML for a ToC entry by line number
 * This is needed because visibleTableOfContents has different indices than tableOfContents
 */
function getTocEntryHTML(line: number): string {
  if (!tableOfContents.value) return ''
  const idx = tableOfContents.value.findIndex(e => e.line === line)
  if (idx === -1) return ''
  return tocEntryHTML.value[idx] || ''
}

/**
 * Get visible ToC entries based on collapse state AND search filter
 */
const filteredTableOfContents = computed(() => {
  if (!tableOfContents.value) return []
  
  const result: any[] = []
  const collapsedLevels: number[] = [] // Stack of collapsed heading levels
  const query = searchQuery.value.toLowerCase().trim()
  
  for (let i = 0; i < tableOfContents.value.length; i++) {
    const entry = tableOfContents.value[i]
    
    // Check if this entry should be hidden due to a collapsed parent
    let shouldHide = false
    for (const collapsedLevel of collapsedLevels) {
      if (entry.level > collapsedLevel) {
        shouldHide = true
        break
      }
    }
    
    // Check if this entry matches search query
    const matchesSearch = query.length === 0 || entry.text.toLowerCase().includes(query)
    
    // Show entry if not hidden by collapse AND matches search (or no search)
    if (!shouldHide && (matchesSearch || query.length === 0)) {
      result.push(entry)
    }
    
    // Update collapsed levels stack
    const isEntryCollapsed = collapsedHeadings.value.has(entry.line)
    
    // If this entry is collapsed and has children, add its level to the stack
    if (isEntryCollapsed && hasChildren(entry)) {
      collapsedLevels.push(entry.level)
    }
    
    // Clean up stack: remove levels when we move to a sibling or parent
    // Look ahead to next entry
    const hasNext = i < tableOfContents.value.length - 1
    if (hasNext) {
      const nextEntry = tableOfContents.value[i + 1]
      // Pop all levels that are >= next entry's level
      while (collapsedLevels.length > 0 && collapsedLevels[collapsedLevels.length - 1] >= nextEntry.level) {
        collapsedLevels.pop()
      }
    } else {
      // Last entry, clear the stack
      collapsedLevels.length = 0
    }
  }
  
  return result
})

/**
 * Check the last "collapse all" action to determine button icon
 * This is INDEPENDENT of manual collapse/expand operations
 * false = last action was "expand all" → show 📁 (click to collapse all)
 * true = last action was "collapse all" → show 📂 (click to expand all)
 */
const allCollapsed = computed(() => {
  return lastCollapseAllState.value
})

/**
 * Returns either the title property for the active file or the generic ToC
 * label -- to be used within the ToC of the sidebar
 *
 * @return  {string}  The title for the ToC sidebar
 */
const titleOrTocLabel = computed(() => {
  if (
    activeFileDescriptor.value === null ||
    activeFileDescriptor.value.type !== 'file' ||
    activeFileDescriptor.value.frontmatter == null
  ) {
    return trans('Table of contents')
  }

  const frontmatter = activeFileDescriptor.value.frontmatter

  if ('title' in frontmatter && frontmatter.title.length > 0) {
    return frontmatter.title
  } else {
    return trans('Table of contents')
  }
})

const activeFile = computed(() => documentTreeStore.lastLeafActiveFile)

watch(activeFile, async (newValue) => {
  if (newValue === undefined) {
    activeFileDescriptor.value = null
  } else {
    const descriptor: AnyDescriptor|undefined = await ipcRenderer.invoke('application', {
      command: 'get-descriptor',
      payload: newValue.path
    })

    activeFileDescriptor.value = descriptor ?? null
  }
})

watch(activeFileDescriptor, (newValue) => {
  if (newValue === null || newValue.type !== 'file') {
    library.value = CITEPROC_MAIN_DB
  } else {
    const fm = newValue.frontmatter
    if (fm != null && 'bibliography' in fm && typeof fm.bibliography === 'string' && fm.bibliography.length > 0) {
      library.value = fm.bibliography
    }
  }
})

/**
 * Whether the cursor is within the corresponding document section
 *
 * @param   {number}  tocEntryLine          Line number of section heading
 */
function tocEntryIsActive (tocEntryLine: number): boolean {
  // If auto-scroll is disabled, don't highlight active entry
  if (!autoScrollEnabled.value) return false
  
  if (tableOfContents.value === undefined || windowStateStore.activeDocumentInfo === undefined) {
    return false
  }

  const cursorLine = windowStateStore.activeDocumentInfo.cursor.line

  // Find the index of current entry in the ORIGINAL tableOfContents
  const currentIdx = tableOfContents.value.findIndex(e => e.line === tocEntryLine)
  if (currentIdx === -1) return false

  // Determine index of next heading in ToC list
  const nextTocEntryIdx = Math.min(currentIdx + 1, tableOfContents.value.length - 1)

  // Now, determine the next heading's line number
  let nextTocEntryLine = Infinity
  if (currentIdx !== nextTocEntryIdx) {
    nextTocEntryLine = tableOfContents.value[nextTocEntryIdx].line
  }

  // True, when cursor lies between current and next heading
  return (cursorLine >= tocEntryLine && cursorLine < nextTocEntryLine)
}

/**
 * Converts the ToC entries's texts to (safe) HTML.
 */
function updateToCHTML () {
  if (tableOfContents.value === undefined) {
    return
  }

  const promises: Promise<string>[] = []

  for (const entry of tableOfContents.value) {
    promises.push(
      md2html(entry.text, {
        onCitation: window.getCitationCallback(library.value),
        zknLinkFormat: configStore.config.zkn.linkFormat,
        sanitizeHTML: true
      })
    )
  }

  Promise.all(promises)
    .then(values => {
      values = values.map(html => {
        return sanitizeHtml(html, {
          // Headings may be emphasised and contain code
          allowedTags: [ 'em', 'kbd', 'code' ]
        })
      })

      tocEntryHTML.value = values
    })
    .catch(err => console.error(err))
}

function startDragging (event: DragEvent): void {
  if (event.currentTarget === null) {
    return
  }
  const fromLine = (event.currentTarget as HTMLElement).dataset.line
  if (fromLine !== undefined) {
    event.dataTransfer?.setData('x-zettlr/toc-drag', fromLine)
  }
}

function dragOver (event: DragEvent): void {
  const elem = document.querySelectorAll('.toc-entry-container')
  elem.forEach(e => e.classList.remove('toc-drop-effect'))
  const container = event.currentTarget as HTMLElement
  container.classList.add('toc-drop-effect')
}

function drop (event: DragEvent): void {
  if (event.currentTarget === null || event.dataTransfer === null) {
    return
  }

  const container = event.currentTarget as HTMLElement
  container.classList.remove('toc-drop-effect')

  if (container.dataset.line === undefined) {
    return
  }

  const fromLine = parseInt(event.dataTransfer.getData('x-zettlr/toc-drag'), 10)
  const toLine = parseInt(container.dataset.line, 10)
  if (fromLine === toLine) {
    return
  }

  const actualToLine = findEndOfEntry(toLine)
  if (actualToLine === undefined) {
    console.warn('Could not move section: Could not find correct target line')
    return
  }

  emit('move-section', { from: fromLine, to: actualToLine })
}

function findEndOfEntry (originalToLine: number): number|undefined {
  if (tableOfContents.value == null) {
    return
  }

  const idx = tableOfContents.value.findIndex(elem => elem.line === originalToLine)

  if (idx < 0) {
    return
  }

  if (idx === tableOfContents.value.length - 1) {
    return -1
  } else {
    return tableOfContents.value[idx + 1].line
  }
}
</script>

<style lang="less">
// Toolbar
.toc-toolbar {
  display: flex;
  gap: 8px;
  margin-bottom: 12px;
  padding: 8px 0;
  flex-wrap: wrap;
}

.toc-search {
  flex: 1;
  min-width: 100px;
  padding: 4px 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 12px;
  background: #fff;
  color: #333;
  height: 28px;
  
  &:focus {
    outline: none;
    border-color: #0066cc;
    box-shadow: 0 0 0 2px rgba(0, 102, 204, 0.15);
  }
  
  &::placeholder {
    color: #999;
  }
}

.toc-toolbar-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  padding: 0;
  border: 1px solid #ccc;
  border-radius: 4px;
  background: #f5f5f5;
  color: #333;
  cursor: pointer;
  transition: all 0.15s ease;
  flex-shrink: 0;
  
  &:hover {
    background: #e8e8e8;
    border-color: #999;
  }
  
  &.active {
    background: #0066cc;
    border-color: #0066cc;
    color: #fff;
  }
  
  .btn-icon {
    font-size: 16px;
    line-height: 1;
  }
}

// Add a neat little effect to the table of content entries as you drag them
.toc-entry-container {
  border-bottom: 2px solid transparent;
  display: flex;
  align-items: center;
  position: relative;
  min-height: 24px;

  &.toc-heading-1 { margin-left: 0px; }
  &.toc-heading-2 { margin-left: 10px; }
  &.toc-heading-3 { margin-left: 20px; }
  &.toc-heading-4 { margin-left: 30px; }
  &.toc-heading-5 { margin-left: 40px; }
  &.toc-heading-6 { margin-left: 50px; }

  &.toc-drop-effect {
    border-bottom-color: rgb(40, 100, 255);
  }

  &.toc-entry-active {
    background-color: rgba(0, 102, 204, 0.1);
    border-left: 3px solid #0066cc;
    padding-left: 4px;
  }
}

// Collapse area: always takes up space for alignment
.toc-collapse-area {
  width: 22px;
  height: 22px;
  margin-right: 2px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

// Collapse/Expand toggle button (visible for entries with children)
.toc-collapse-toggle {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 18px;
  height: 18px;
  cursor: pointer;
  opacity: 0.6;
  transition: opacity 0.15s ease, background-color 0.15s ease, transform 0.2s ease;
  user-select: none;
  border-radius: 3px;

  &:hover {
    opacity: 1;
    background-color: rgba(0, 0, 0, 0.08);
  }

  .toggle-icon {
    display: inline-block;
    font-size: 10px;
    color: #666;
    line-height: 1;
    transition: transform 0.2s ease;
  }
  
  // Rotate icon when collapsed (pointing right)
  &.collapsed .toggle-icon {
    transform: rotate(-90deg);
  }
  
  // Expanded state (pointing down, default)
  &.expanded .toggle-icon {
    transform: rotate(0deg);
  }
  
  // Use downward triangle as base icon
  .toggle-icon::before {
    content: '▼';
  }
}

.toc-level {
  margin-right: 6px;
  opacity: 0.6;
  font-size: 0.85em;
  min-width: 20px;
  text-align: right;
}

.toc-entry {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
</style>
