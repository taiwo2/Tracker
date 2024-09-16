<template>
  <div>
    <!-- Modal Overlay -->
    <div
      v-if="dialog"
      class="fixed inset-0 bg-gray-500 bg-opacity-50 flex items-center justify-center z-50"
    >
      <div class="bg-white p-6 rounded-lg shadow-lg w-full max-w-md">
        <div class="flex items-center justify-between mb-4">
          <h3 class="text-lg font-bold">Add Board</h3>
          <button @click="closeDialog" class="text-gray-600 hover:text-gray-900">
            &times;
          </button>
        </div>
        <form @submit.prevent="createBoard">
          <div class="flex flex-col space-y-4">
            <input
              type="text"
              v-model="board.title"
              placeholder="Board title"
              class="border p-2 rounded"
              required
            />
            <button
              v-if="!enableColor"
              type="button"
              @click="enableColor = true"
              class="bg-gray-300 text-gray-800 px-4 py-2 rounded"
            >
              Choose board color
            </button>
            <input
              v-else
              type="color"
              v-model="board.color"
              class="w-full h-10 rounded cursor-pointer"
            />
            <div
              class="flex flex-col items-center justify-center cursor-pointer border-2 border-dashed border-gray-300 p-6 mb-4 rounded-lg"
              @click="chooseImage"
              :style="{ backgroundImage: `url(${imagePreview || board.image.downloadURL || ''})`, height: '150px', backgroundSize: 'cover', backgroundPosition: 'center' }"
            >
              <template v-if="!imagePreview && !board.image.downloadURL">
                <svg class="w-6 h-6 text-gray-700" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4.5m0 0H8.5m3.5 0h3.5M15 12l-3 3-3-3m6 0l-3-3-3 3" />
                </svg>
                <p class="text-gray-700">Add a board background</p>
              </template>
              <input
                type="file"
                accept="image/*"
                ref="boardBackground"
                @change="onFileSelected"
                class="hidden"
              />
            </div>
            <button
              type="submit"
              :disabled="!board.title.trim()"
              :class="[
                'px-4 py-2 rounded',
                board.title.trim() ? 'bg-blue-500 text-white' : 'bg-gray-300 text-gray-500 cursor-not-allowed'
              ]"
            >
              Submit
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Main Content -->
    <div class="flex items-center justify-between mb-4 p-6">
      <h1 class="text-2xl font-bold">My Boards</h1>
      <button @click="addBoard" class="bg-gray-300 text-gray-800 px-4 py-2 rounded">
        ADD BOARD
      </button>
    </div>

    <div class="flex flex-wrap items-center justify-start">
      <p v-if="boards.length === 0" class="text-gray-500">You have no boards yet.</p>
      <div
        class="bg-white p-4 m-2 rounded-lg shadow-md cursor-pointer"
        v-for="board in boards"
        :key="board.id"
        :style="{ backgroundImage: `url(${board.image.downloadURL || ''})`, backgroundColor: board.color || '' }"
        @click="openBoard(board.id)"
      >
        <h2 class="text-lg font-semibold" :style="{ color: board.image.downloadURL ? '#fff' : '' }">
          {{ board.title }}
        </h2>
        <p class="text-sm text-gray-600" :style="{ color: board.image.downloadURL ? '#fff' : '' }">
          created {{ formatDate(board.dateCreated) }}
        </p>
      </div>
    </div>

    <div
      v-if="snackbar"
      class="fixed bottom-0 left-1/2 transform -translate-x-1/2 bg-purple-200 text-purple-800 px-4 py-2 rounded shadow-lg"
    >
      {{ snackbarText }}
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue'
import { navigateTo } from '#app'

export default defineComponent({
  data() {
    return {
      enableColor: false,
      dialog: false,
      imagePreview: null,
      board: {
        title: '',
        color: '#ffffff',
        image: {
          name: '',
          originalName: '',
          downloadURL: '',
          uuid: '',
        },
      },
      snackbar: false,
      snackbarText: '',
      boards: [],
    }
  },
  mounted() {
    this.loadBoards()
  },
  methods: {
    loadBoards() {
      const storedBoards = JSON.parse(localStorage.getItem('boards') || '[]')
      this.boards = storedBoards
      console.log('Loaded boards from localStorage:', this.boards)
    },
    addBoard() {
      this.dialog = true
    },
    createBoard() {
      if (this.board.title.trim()) {
        const newBoard = {
          id: this.generateId(),
          title: this.board.title,
          color: this.board.color,
          image: { ...this.board.image },
          dateCreated: Date.now()
        }
        
        this.boards.push(newBoard)
        this.saveBoards()
        
        this.closeDialog()
        
        this.snackbarText = 'Successfully created your board'
        this.snackbar = true
        
        console.log('New board created:', newBoard)
        console.log('Current boards:', this.boards)
      }
    },
    closeDialog() {
      this.dialog = false
      this.resetForm()
    },
    resetForm() {
      this.board = {
        title: '',
        color: '#ffffff',
        image: {
          name: '',
          originalName: '',
          downloadURL: '',
          uuid: '',
        },
      }
      this.imagePreview = null
      this.enableColor = false
    },
    saveBoards() {
      localStorage.setItem('boards', JSON.stringify(this.boards))
    },
    formatDate(value) {
      if (!value) return ''
      const date = new Date(value)
      return date.toLocaleString()
    },
    chooseImage() {
      this.$refs.boardBackground.click()
    },
    onFileSelected(event) {
      const file = event.target.files[0]
      if (file) {
        const reader = new FileReader()
        reader.onload = (e) => {
          this.imagePreview = e.target.result
          this.board.image = {
            name: file.name,
            originalName: file.name,
            downloadURL: e.target.result,
            uuid: this.generateId(),
          }
        }
        reader.readAsDataURL(file)
      }
    },
    openBoard(id) {
      console.log(`Navigating to board with ID: ${id}`)
      return navigateTo(`/boards/${id}`)
    },
    generateId() {
      return '_' + Math.random().toString(36).substr(2, 9)
    }
  }
})
</script>