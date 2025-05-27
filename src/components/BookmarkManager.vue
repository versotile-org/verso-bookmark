<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Bookmark {
  url: string
  name: string
  isRenaming: boolean
}

const bookmarks = ref<Bookmark[]>([])

onMounted(() => {
  requestBookmark()
})

const requestBookmark = () => {
  const response = window.prompt('VERSO::BOOKMARK_LIST_GET')
  if (response !== null) {
    const status = JSON.parse(response) as { [url: string]: Bookmark }
    bookmarks.value = Object.values(status)
  }
}

const onclickRemove = (id: string) => {
  if (bookmarks.value.some(b => b.isRenaming)){
    return
  }
  window.prompt('VERSO::BOOKMARK_REMOVE::' + id)
}

const endRename = (id: string, name: string) => {
  const bookmark = bookmarks.value.find(b => b.url === id)
  if (bookmark) {
    bookmark.isRenaming = false
    window.prompt('VERSO::BOOKMARK_RENAME::' + id + '::' + name)
  }
}
const onclickStartRename = (id: string) => {
  if (bookmarks.value.some(b => b.isRenaming)) {
    return
  }
  const bookmark = bookmarks.value.find(b => b.url === id)
  if (bookmark) {
    bookmark.isRenaming = true
  }
}
const onclickNavigate = (url: string) => {
  window.prompt(`VERSO::NAVIGATE_TO::${url}`);
}

setInterval(() => {
  // if no bookmark is renaming, request bookmarks
  if (!bookmarks.value.some(b => b.isRenaming)) {
    requestBookmark()
  }
}, 2000)
</script>

<template>
  <div class="bookmark-manager">
    <h2 class="bookmark-title">Bookmark Manager</h2>
    <div class="bookmark-container">
      <div v-for="bookmark in bookmarks" :key="bookmark.url" class="bookmark-item">
        <div class="name-url-row">
          <div v-if="!bookmark.isRenaming" class="name" @click="onclickNavigate(bookmark.url)">
            <a :href="bookmark.url" v-text="bookmark.name"></a>
          </div>
          <input v-if="bookmark.isRenaming" @keyup.enter="endRename(bookmark.url, bookmark.name)" class="rename-input" v-model="bookmark.name">
          <div class="url">{{ bookmark.url }}</div>
        </div>
        <div class="status-row">
          <button @click="onclickStartRename(bookmark.url)" class="action-btn" :class="{'inactive' : bookmarks.some(b => b.isRenaming)}">
            Rename
          </button>
          <button @click="onclickRemove(bookmark.url)" class="action-btn" :class="{'inactive' : bookmarks.some(b => b.isRenaming)}">
            Remove
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.bookmark-manager {
  max-width: 800px;
  margin: 0 auto;
  padding: 1rem;
}

.bookmark-container {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.bookmark-title {
  font-size: 2rem;
  font-weight: bold;
  margin-bottom: 1rem;
}

.bookmark-item {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  padding: 0.75rem;
  background-color: #2a2a2a;
  border-radius: 0.5rem;
  color: #ffffff;

  .name-url-row {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    gap: 1rem;
  }

  .name {
    flex: 1;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  a {
    color: #ffffff
  }

  .url {
    flex: 1;
    font-size: 0.875rem;
    color: #9ca3af;
    text-align: right;
  }

  .status-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .status {
    color: #9ca3af;
    text-transform: capitalize;
    font-size: 0.875rem;
  }

  .action-btn {
    padding: 0.25rem 0.75rem;
    background-color: #dc2626;
    color: white;
    border: none;
    border-radius: 0.25rem;
    cursor: pointer;
    font-size: 0.875rem;
    transition: background-color 0.2s;
  }
  .inactive {
    background-color: #4b5563;
    cursor: not-allowed;
  }
}
</style>
