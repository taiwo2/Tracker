<template>
  <div v-if="error">
    <p>Error loading board: {{ error.message }}</p>
  </div>
  <div
    v-else-if="board"
    class="flex flex-col board"
    :style="
      board.image && board.image.downloadURL
        ? `background:url(${board.image.downloadURL});background-size:cover;`
        : board.color
        ? `background-color:${board.color}`
        : ''
    "
  >

    <main class="p-6">
    <h1 class="text-2xl font-bold">{{ board.title }}</h1>
    <small class="text-gray-500">Created {{ new Date(board.dateCreated).toLocaleDateString() }}</small>

    <!-- Flex container for lists -->
    <div class="flex flex-wrap space-x-4 mt-6">
      <div
        v-for="list in board.lists"
        :key="list.id"
        class="bg-gray-100 p-4 rounded-lg w-64 shadow-sm relative"
        @drop="drop($event, list.id)"
        @dragover.prevent
      >
        <div class="flex justify-between items-center mb-4">
          <h4 class="text-lg font-semibold">{{ list.title }}</h4>
          <button @click="deleteList(list.id)" class="text-gray-400 hover:text-red-500">
            <TrashIcon class="w-5 h-5" />
          </button>
        </div>

        <!-- Cards section -->
        <div v-for="card in list.cards" :key="card.id" 
          class="bg-white p-2 mb-3 rounded-lg shadow cursor-pointer"
          draggable="true" @dragstart="drag($event, card)"
          @click="openCardModal(card)">
          <p class="text-sm text-gray-700">{{ card.title }}</p>
        </div>

        <!-- Add card button -->
        <button @click="addCard(list.id)" class="flex items-center text-blue-500 hover:text-blue-600 mt-4">
          <PlusIcon class="w-5 h-5" />
          <span class="ml-1 text-sm">Add a card</span>
        </button>
      </div>
    </div>

    <!-- Create new list button -->
    <button @click="openCreateListModal" class="mt-6 bg-green-500 text-white px-4 py-2 rounded shadow hover:bg-green-600">
      Create new list
    </button>

    <!-- Modal for creating a new list -->
    <div v-if="isCreateListModalOpen" class="fixed inset-0 bg-gray-500 bg-opacity-75 flex items-center justify-center z-50">
      <div class="bg-white rounded-lg p-6 w-96 shadow-lg">
        <h2 class="text-xl font-semibold mb-4">Create New List</h2>
        <input 
          v-model="newListTitle" 
          placeholder="Enter list title" 
          class="w-full border border-gray-300 p-2 rounded mb-4" 
          type="text" 
        />
        <div class="flex justify-end">
          <button @click="isCreateListModalOpen = false" class="mr-2 bg-gray-300 text-gray-700 px-4 py-2 rounded">
            Cancel
          </button>
          <button @click="createNewList" class="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600">
            Create
          </button>
        </div>
      </div>
    </div>

    <!-- Modal for editing a card -->
    <div v-if="isCardModalOpen" class="fixed inset-0 bg-gray-500 bg-opacity-75 flex items-center justify-center z-50">
      <div class="bg-white rounded-lg p-6 w-96 shadow-lg">
        <h2 class="text-xl font-semibold mb-4">Edit Card</h2>
        <input 
          v-model="selectedCard.title" 
          placeholder="Enter card title" 
          class="w-full border border-gray-300 p-2 rounded mb-4" 
          type="text" 
        />
        <div class="flex justify-between">
          <button @click="deleteCard(selectedCard)" class="flex items-center text-red-500 hover:text-red-600">
            <TrashIcon class="w-5 h-5 mr-1" /> Delete
          </button>
          <div>
            <button @click="closeCardModal" class="mr-2 bg-gray-300 text-gray-700 px-4 py-2 rounded">
              Close
            </button>
            <button @click="saveCardEdit" class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600">
              Save
            </button>
          </div>
        </div>
      </div>
    </div>
  </main>
  </div>
  <div v-else>
    <p>Board not found. ID: {{ boardId }}</p>
    <p>Available boards: {{ boards.length }}</p>
    <ul>
      <li v-for="b in boards" :key="b.id">ID: {{ b.id }}, Title: {{ b.title }}</li>
    </ul>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { TrashIcon, PlusIcon } from '@heroicons/vue/outline'

const route = useRoute()
const router = useRouter()
const boards = ref([])
const error = ref(null)
const drawer = ref(false)
const isCreateListModalOpen = ref(false); // Modal for creating new list
const isCardModalOpen = ref(false); // Modal for editing card
const newListTitle = ref(''); // Holds new list title
const selectedCard = ref(null); // Holds the card being edited

// const emit = defineEmits(['updateBackground'])

// watch(board, (newBoard) => {
//   if (newBoard) {
//     emit('updateBackground', {
//       image: newBoard.image?.downloadURL,
//       color: newBoard.color
//     })
//   }
// }, { immediate: true })
const boardId = computed(() => {
  console.log('Route params:', route.params)
  const id = route.params._id
  console.log('Current board ID:', id)
  return id
})

const board = computed(() => {
  console.log('Looking for board with ID:', boardId.value)
  const foundBoard = boards.value.find(b => b.id === boardId.value)
  console.log('Found board:', foundBoard)
  return foundBoard
})

onMounted(() => {
  loadBoards()
})

function loadBoards() {
  try {
    const storedBoards = JSON.parse(localStorage.getItem('boards') || '[]')
    boards.value = storedBoards
    console.log('Stored boards:', storedBoards)
    console.log('Current board ID:', boardId.value)
    console.log('Found board:', board.value)
    
    if (boardId.value && !board.value) {
      error.value = new Error(`Board with ID ${boardId.value} not found`)
    }
  } catch (e) {
    console.error('Error loading boards:', e)
    error.value = e
  }
}

function saveBoard() {
  const updatedBoards = boards.value.map(b => b.id === board.value.id ? board.value : b)
  localStorage.setItem('boards', JSON.stringify(updatedBoards))
}

function createList() {
  if (board.value) {
    const newList = {
      id: Date.now().toString(),
      title: 'New List',
      cards: [],
    }
    board.value.lists = board.value.lists || []
    board.value.lists.push(newList)
    saveBoard()
  }
}

function addCard(listId) {
  if (board.value) {
    const list = board.value.lists.find((list) => list.id === listId)
    if (list) {
      list.cards.push({
        id: Date.now().toString(),
        title: 'New Card',
      })
      saveBoard()
    }
  }
}

function deleteList(listId) {
  if (board.value) {
    board.value.lists = board.value.lists.filter((list) => list.id !== listId)
    saveBoard()
  }
}

function deleteBoard() {
  const updatedBoards = boards.value.filter(b => b.id !== boardId.value)
  localStorage.setItem('boards', JSON.stringify(updatedBoards))
  router.push('/')
}

function drag(event, card) {
  event.dataTransfer.setData('cardId', card.id)
}

function drop(event, listId) {
  event.preventDefault()
  const cardId = event.dataTransfer.getData('cardId')
  if (board.value) {
    const targetList = board.value.lists.find((list) => list.id === listId)
    let movedCard
    board.value.lists.forEach(list => {
      const cardIndex = list.cards.findIndex(card => card.id === cardId)
      if (cardIndex !== -1) {
        movedCard = list.cards.splice(cardIndex, 1)[0]
      }
    })
    if (movedCard && targetList) {
      targetList.cards.push(movedCard)
      saveBoard()
    }
  }
}

function openCreateListModal() {
  newListTitle.value = ''; // Clear input before opening modal
  isCreateListModalOpen.value = true;
}
  function createNewList() {
  if (newListTitle.value.trim() === '') return; // Don't allow empty lists

  // Initialize board.value and board.value.lists if they're undefined
  if (!board.value) {
    board.value = { lists: [] };
  } else if (!board.value.lists) {
    board.value.lists = [];
  }

  const newList = {
    id: Date.now().toString(), // Simple unique ID
    title: newListTitle.value,
    cards: []
  };

  board.value.lists.push(newList); // Add the new list to the board
  saveBoard(); // Save changes
  isCreateListModalOpen.value = false; // Close the modal
}

function openCardModal(card) {
  selectedCard.value = { ...card }; // Copy the card to the modal state
  isCardModalOpen.value = true;
}

function saveCardEdit() {
  const list = board.value.lists.find(l => l.cards.find(c => c.id === selectedCard.value.id));
  const cardIndex = list.cards.findIndex(c => c.id === selectedCard.value.id);
  list.cards[cardIndex] = { ...selectedCard.value }; // Save the edits
  saveBoard(); // Save changes
  isCardModalOpen.value = false; // Close the modal
}

function deleteCard(card) {
  const list = board.value.lists.find(l => l.cards.find(c => c.id === card.id));
  list.cards = list.cards.filter(c => c.id !== card.id); // Remove the card
  saveBoard(); // Save changes
  isCardModalOpen.value = false; // Close the modal
}

function closeCardModal() {
  isCardModalOpen.value = false;
}

// Watch for changes in the route params
watch(() => route.params._id, (newId) => {
  console.log('Route params changed, new ID:', newId)
  loadBoards()
})

// Add a global error handler
router.onError((error) => {
  console.error('Navigation error:', error)
  error.value = error
})

</script>
