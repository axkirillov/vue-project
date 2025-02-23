<template>
	<div class="w-full">
		<div class="flex items-center justify-center mb-4 relative">
			<div class="logos flex-grow flex justify-center">
				<a href="" class="logo mr-2">
					<img src="/hamster.svg">
				</a>
				<a href="" class="logo">
					<img src="/basket.png">
				</a>
			</div>
		</div>
		<h1 class="m-2 font-semibold text-2xl text-center whitespace-pre-wrap break-all w-full overflow-wrap-anywhere">Hamster's Basket 2.0</h1>
		<nav v-if="allLists.length > 0" class="flex items-center justify-center space-x-2 text-sm text-gray-600 mb-2">
			<div class="relative">
				<button 
					@click="toggleListDropdown" 
					class="flex items-center space-x-1 hover:bg-gray-100 rounded-md px-2 py-1 transition duration-200"
				>
					<span>Lists</span>
					<font-awesome-icon :icon="['fas', 'chevron-right']" class="w-3 h-3" />
					<span class="font-semibold">{{ currentList?.name }}</span>
					<font-awesome-icon 
						:icon="['fas', 'caret-down']" 
						class="w-3 h-3 ml-1 text-gray-500" 
					/>
				</button>
				
				<!-- List Dropdown -->
				<div 
					v-if="listDropdownOpen" 
					class="absolute left-0 mt-2 w-56 origin-top-left bg-white divide-y divide-gray-100 rounded-md shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none z-50"
				>
					<div class="py-1">
						<button 
							v-for="list in allLists" 
							:key="list.id"
							@click="selectList(list); listDropdownOpen = false"
							:class="{
								'bg-blue-500 text-white': currentList?.id === list.id,
								'hover:bg-gray-100': currentList?.id !== list.id
							}"
							class="
								text-left 
								w-full 
								px-4 
								py-2 
								text-sm 
								transition 
								duration-200 
								flex 
								items-center 
								justify-between
							"
						>
							<span>{{ list.name }}</span>
							<font-awesome-icon 
								v-if="currentList?.id === list.id" 
								:icon="['fas', 'check']" 
								class="w-4 h-4" 
							/>
						</button>
						<div class="border-t border-gray-200 py-1">
							<button 
								@click="createListPrompt(); listDropdownOpen = false" 
								class="
									text-gray-900 
									group 
									flex 
									rounded-md 
									items-center 
									w-full 
									px-4 
									py-2 
									text-sm 
									hover:bg-gray-100
								"
							>
								<font-awesome-icon 
									:icon="['fas', 'plus']" 
									class="mr-2 h-5 w-5 text-gray-500"
								/>
								Create New List
							</button>
							<button 
								v-if="currentList && currentList.name !== 'Default List'"
								@click="deleteCurrentList(); listDropdownOpen = false"
								class="
									text-red-600 
									group 
									flex 
									rounded-md 
									items-center 
									w-full 
									px-4 
									py-2 
									text-sm 
									hover:bg-gray-100
								"
							>
								<font-awesome-icon 
									:icon="['fas', 'trash']" 
									class="mr-2 h-5 w-5 text-red-500"
								/>
								Delete Current List
							</button>
						</div>
					</div>
				</div>
			</div>
			<!-- Flyout Menu -->
			<div 
				v-if="sideMenuOpen" 
				class="absolute left-0 mt-2 w-56 origin-top-left bg-white divide-y divide-gray-100 rounded-md shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none z-50"
			>
				<div class="py-1">
					<button 
						@click="createListPrompt(); sideMenuOpen = false" 
						class="text-gray-900 group flex rounded-md items-center w-full px-4 py-2 text-sm hover:bg-gray-100"
					>
						<font-awesome-icon 
							:icon="['fas', 'plus']" 
							class="mr-2 h-5 w-5 text-gray-500"
						/>
						Create New List
					</button>
					<button 
						@click="toggleListSelection"
						class="text-gray-900 group flex rounded-md items-center w-full px-4 py-2 text-sm hover:bg-gray-100"
					>
						<font-awesome-icon 
							:icon="['fas', 'list']" 
							class="mr-2 h-5 w-5 text-gray-500"
						/>
						Select List
					</button>
					<button 
						@click="deleteCurrentList(); sideMenuOpen = false"
						class="text-red-600 group flex rounded-md items-center w-full px-4 py-2 text-sm hover:bg-gray-100"
					>
						<font-awesome-icon 
							:icon="['fas', 'trash']" 
							class="mr-2 h-5 w-5 text-red-500"
						/>
						Delete Current List
					</button>
				</div>
			</div>
		</nav>

		<!-- List Selection Overlay -->
		<div 
			v-if="showListSelection" 
			class="fixed inset-0 bg-black bg-opacity-50 z-50 flex items-center justify-center"
			@click.self="showListSelection = false"
		>
			<div class="bg-white rounded-lg p-6 max-w-md w-full max-h-[80vh] overflow-y-auto">
				<h2 class="text-xl font-semibold mb-4">Select a List</h2>
				<div class="space-y-2">
					<button 
						v-for="list in allLists" 
						:key="list.id"
						@click="selectList(list); showListSelection = false"
						:class="{
							'bg-blue-500 text-white': currentList?.id === list.id,
							'bg-gray-200 hover:bg-gray-300': currentList?.id !== list.id
						}"
						class="w-full text-left px-4 py-2 rounded-md transition duration-200"
					>
						{{ list.name }}
					</button>
				</div>
			</div>
		</div>

		<div class="bg-white shadow overflow-hidden rounded-md py-2">
			<ul v-if="currentList">
				<Todo v-for="(todo, index) in filteredTodos" :key="index" :todo="todo" />
			</ul>
			<p v-else class="text-center text-gray-500">
				Create a list to get started!
			</p>
		</div>
		<div class="flex gap-2 my-4">
			<input
				v-model="task"
				class="
					rounded 
					w-full 
					p-2 
					whitespace-pre-wrap 
					break-all 
					overflow-wrap-anywhere
				"
				type="text"
				placeholder="What do you need to do?"
				@keyup.enter="insertTask"
			/>
			<button @click="insertTask" class="btn-black">
				Add
			</button>
		</div>
	</div>
</template>

<script lang="ts">
/* eslint-disable @typescript-eslint/camelcase */
import { defineComponent, ref, onMounted, computed } from 'vue'
import Todo from '@/components/Todo.vue'
import { supabase } from '@/lib/supabase'
import { allTodos, allLists, fetchTodos, fetchLists, addTodo, addList, currentList, deleteList } from '@/vuetils/useTodo'
import { userSession } from '@/vuetils/useAuth'

// Ensure existing todos are migrated to the default list
async function migrateExistingTodos(defaultListId: number) {
  const existingTodos = await supabase.from('todos').select('*').is('list_id', null)
  
  if (existingTodos.data && existingTodos.data.length > 0) {
    const updatePromises = existingTodos.data.map(todo => 
      supabase.from('todos').update({ list_id: defaultListId }).eq('id', todo.id)
    )
    
    await Promise.all(updatePromises)
  }
}

export default defineComponent({
	name: 'TodoList',
	components: {
		Todo,
	},

	async setup() {
		const task = ref('')
		const listName = ref('')

		onMounted(async () => {
			await fetchLists()
			if (currentList.value && currentList.value.id) {
				await fetchTodos(currentList.value.id)
				
				// Migrate existing todos to the default list if needed
				if (currentList.value.name === 'Default List') {
					await migrateExistingTodos(currentList.value.id)
					await fetchTodos(currentList.value.id)
				}
			}
		})

		async function createList() {
			if (!listName.value.trim()) {
				alert('Please provide a list name')
				return
			}

			if (userSession?.value === null) {
				alert('Please log in again')
				return
			}

			const newList = await addList({ 
				user_id: userSession.value.user.id, 
				name: listName.value 
			})

			if (newList) {
				listName.value = ''
				await fetchLists()
			}
		}

		function selectList(list: TodoList) {
			currentList.value = list
			fetchTodos(list.id)
		}

		/**
		 * Wrapper function adding a new todo for additional client-side error handling.
		 */
		async function insertTask() {
			// Guard for short task descriptions which will fail db policy.
			if (!task.value.trim()) {
				alert('Please enter a task')
				return
			}
			// Type check to ensure user is still logged in.
			if (userSession?.value === null) {
				alert('Please log in again')
				return
			}
			try {
				// Try and write the data to the database.
				const todo = await addTodo({ 
				  user_id: userSession.value.user.id, 
				  task: task.value, 
				  list_id: currentList.value?.id ?? undefined 
				})

				// If there was no response, don't do anything.
				if (!todo) {
					return
				}
				// Otherwise, push the response into allTodos.
				allTodos.value.push(todo)

				// Reset input field.
				task.value = ''
			} catch (err) {
				console.error('Unknown error when adding todo', err)
			}
		}

		const filteredTodos = computed(() => 
			allTodos.value.filter(todo => todo.list_id === currentList.value?.id)
		)

		async function confirmDeleteList(list: TodoList) {
			const confirmDelete = confirm(`Are you sure you want to delete the list "${list.name}"? All todos in this list will be permanently deleted.`)
			
			if (confirmDelete) {
				const success = await deleteList(list)
				if (success) {
					await fetchLists()
				}
			}
		}

		const sideMenuOpen = ref(false)
		const showListSelection = ref(false)
		const listDropdownOpen = ref(false)
		const tabContainer = ref<HTMLDivElement | null>(null)
		const tabScrollPosition = ref(0)
		const tabNavContainer = ref<HTMLDivElement | null>(null)
		const visibleLists = computed(() => {
			// If refs are not ready, return all lists
			if (!tabContainer.value || !tabNavContainer.value) return allLists.value

			const containerWidth = tabContainer.value.clientWidth
			const visibleListsArray = []

			// If no lists, return empty array
			if (allLists.value.length === 0) return []

			// Fallback to all lists if container width is very small
			if (containerWidth < 100) return allLists.value

			for (const list of allLists.value) {
				visibleListsArray.push(list)
				// Optional: Add a break condition if needed
				if (visibleListsArray.length >= 5) break
			}

			return visibleListsArray
		})

		const isMaxScrollReached = computed(() => {
			if (!tabContainer.value) return true
			return tabScrollPosition.value >= tabContainer.value.scrollWidth - tabContainer.value.clientWidth
		})

		function scrollTabs(direction: 'left' | 'right') {
			if (!tabContainer.value) return

			const scrollAmount = tabContainer.value.clientWidth / 2
			tabScrollPosition.value = direction === 'left' 
				? Math.max(0, tabScrollPosition.value - scrollAmount)
				: Math.min(
					tabContainer.value.scrollWidth - tabContainer.value.clientWidth, 
					tabScrollPosition.value + scrollAmount
				)
		}

		function toggleSideMenu() {
			sideMenuOpen.value = !sideMenuOpen.value
		}

		function toggleListSelection() {
			showListSelection.value = !showListSelection.value
			sideMenuOpen.value = false
		}

		function toggleListDropdown() {
			listDropdownOpen.value = !listDropdownOpen.value
		}

		// Click outside handler for side menu and list dropdown
		onMounted(() => {
			const handleClickOutside = (event: MouseEvent) => {
				const menuButton = event.target as HTMLElement
				const sideMenu = document.querySelector('.absolute.left-0.mt-2')
				const listDropdown = document.querySelector('.absolute.left-0.mt-2 + div')
				
				if (sideMenu && !sideMenu.contains(menuButton) && !menuButton.closest('button')) {
					sideMenuOpen.value = false
				}

				if (listDropdown && !listDropdown.contains(menuButton) && !menuButton.closest('button[data-list-dropdown]')) {
					listDropdownOpen.value = false
				}
			}

			document.addEventListener('click', handleClickOutside)

			return () => {
				document.removeEventListener('click', handleClickOutside)
			}
		})

		function deleteCurrentList() {
			if (currentList.value && currentList.value.name !== 'Default List') {
				confirmDeleteList(currentList.value)
			} else {
				alert('Cannot delete the default list')
			}
		}

		function createListPrompt() {
			const newListName = prompt('Enter a name for the new list:')
			if (newListName) {
				listName.value = newListName
				createList()
			}
		}

		return {
			task,
			listName,
			allTodos,
			filteredTodos,
			allLists,
			currentList,
			insertTask,
			createList,
			createListPrompt,
			selectList,
			confirmDeleteList,
			userSession,
			sideMenuOpen,
			toggleSideMenu,
			deleteCurrentList,
			showListSelection,
			toggleListSelection,
			tabContainer,
			tabNavContainer,
			tabScrollPosition,
			scrollTabs,
			isMaxScrollReached,
			visibleLists,
			listDropdownOpen,
			toggleListDropdown,
		}
	},
})
</script>
