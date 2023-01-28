<template>
    <div class="w-full bg-blue-900 overflow-hidden min-h-screen font-[Poppins]">
        <div
            class="mx-4 space-y-2 md:mx-0 md:w-3/4 lg:w-1/2 grid grid-cols-1 rounded-2xl bg-white px-12 py-12 my-24 md:mx-auto">

            <div class="col-span-1 space-y-3">
                <h1 class="text-4xl font-extrabold text-blue-800 text-center mb-10 shadow-md py-2">{{ theme }}</h1>
                <div v-if="toDoList.length" class="w-full relative items-center">
                    <input
                        type="text"
                        placeholder="Search..."
                        class="w-full px-6 py-1 border-2 rounded-2xl bg-gray-200 focus:bg-gray-100
                        focus:placeholder-blue-500 outline-none focus:border-blue-400 duration-500
                        placeholder:text-gray-500"
                        v-model.trim="search"
                        @keyup="searchList"
                    >
                    <MagnifyingGlassIcon @click="searchList"
                                         class="w-6 h-6 text-gray-500 absolute right-2 -mt-7 cursor-pointer hover:text-blue-500"/>
                </div>

                <div class="w-full flex">
                    <input
                        type="text"
                        placeholder="Add tasks..."
                        class="w-full px-6 py-3 border-2 border-r-0 rounded-2xl rounded-r-none outline-none
                        focus:border-blue-400 duration-500 placeholder:text-gray-500"
                        :class="currentToDoId !== null ? 'focus:border-green-400' : ''"
                        v-model.trim="value"
                        @keyup.enter="addOrUpdate"
                    >
                    <button
                        class="cursor-pointer border-2 border-l-0 border-blue-500 px-6 py-3 font-medium text-white rounded-2xl
                        rounded-l-none bg-blue-500 hover:bg-blue-400 hover:border-blue-400 duration-500"
                        v-if="currentToDoId === null"
                        @click="addTask"
                    >
                        Add
                    </button>
                    <button
                        class="cursor-pointer border-2 border-l-0 border-green-500 px-6 py-3 font-medium text-white rounded-2xl
                        rounded-l-none bg-green-500 hover:bg-green-400 hover:border-green-400 duration-500"
                        v-else
                        @click="updateTask"
                    >
                        Edit
                    </button>
                </div>
            </div>


            <div v-if="search.length" class="col-span-1 space-y-2">
                <div
                    v-for="(task, index) in searchListToDo" :key="task.id"
                    class="flex justify-between items-center px-6 py-2 bg-blue-500 rounded-2xl text-white shadow
                    space-x-2 bg-gradient-to-r from-cyan-500 to-blue-500"
                >
                    <div class="flex space-x-2 items-center">
                        <input
                            type="checkbox"
                            class="w-5 h-5 text-start text-green-600 bg-white cursor-pointer hover:bg-green-200
                                hover:text-green-700 duration-500 rounded outline-0"
                            :checked="task.checked"
                            v-model="task.checked"
                            @change="checkArray"
                        />
                        <span class="text-xl">{{ index + 1 + '.' }}</span>
                        <span
                            :class="task.checked ? 'line-through' : ''"
                            class="text-xl">{{ task.title }}
                        </span>
                    </div>
                    <div class="flex space-x-2">
                        <span
                            class="text-end cursor-pointer hover:text-amber-500 duration-500"
                            @click="editTask(task)"
                        >
                            <PencilSquareIcon class="w-6 h-6"/>
                        </span>
                        <span
                            class="text-end cursor-pointer hover:text-amber-500 duration-500"
                            @click="removeTask(index)"
                        >
                            <XMarkIcon class="w-6 h-6"/>
                        </span>
                    </div>
                </div>
            </div>
            <div v-else-if="toDoList.length && tabActive === null" class="col-span-1 space-y-2">
                <div
                    v-for="(task, index) in toDoList" :key="task.id"
                    class="flex justify-between items-center px-6 py-2 bg-blue-500 rounded-2xl text-white shadow
                    space-x-2 bg-gradient-to-r from-cyan-500 to-blue-500"
                >
                    <div class="flex space-x-2 items-center">
                        <input
                            type="checkbox"
                            class="w-5 h-5 text-start text-green-600 bg-white cursor-pointer hover:bg-green-200
                                hover:text-green-700 duration-500 rounded outline-0"
                            :checked="task.checked"
                            v-model="task.checked"
                            @change="checkArray"
                        />
                        <span class="text-xl">{{ index + 1 + '.' }}</span>
                        <span
                            :class="task.checked ? 'line-through' : ''"
                            class="text-xl">{{ task.title }}
                        </span>
                    </div>
                    <div class="flex space-x-2">
                        <span
                            class="text-end cursor-pointer hover:text-amber-500 duration-500"
                            @click="editTask(task)"
                        >
                            <PencilSquareIcon class="w-6 h-6"/>
                        </span>
                        <span
                            class="text-end cursor-pointer hover:text-amber-500 duration-500"
                            @click="removeTask(index)"
                        >
                            <XMarkIcon class="w-6 h-6"/>
                        </span>
                    </div>
                </div>
            </div>

            <div v-else-if="acceptedToDo.length && tabActive === true" class="col-span-1 space-y-2">
                <div
                    v-for="(task, index) in acceptedToDo" :key="task.id"
                    class="flex justify-between items-center px-6 py-2 bg-blue-500 rounded-2xl text-white shadow
                    space-x-2 bg-gradient-to-r from-cyan-500 to-blue-500"
                >
                    <div class="flex space-x-2 items-center">
                        <input
                            type="checkbox"
                            class="w-5 h-5 text-start text-green-600 bg-white cursor-pointer hover:bg-green-200
                                hover:text-green-700 duration-500 rounded"
                            v-if="task.checked"
                            v-model="task.checked"
                            @change="checkArray"
                        />
                        <input
                            type="checkbox"
                            class="w-5 h-5 text-start text-red-600 bg-white cursor-pointer hover:bg-red-200
                                hover:text-red-700 duration-500 rounded"
                            v-else
                            v-model="task.checked"
                            @change="checkArray"
                        />
                        <span class="text-xl">{{ index + 1 + '.' }}</span>
                        <span
                            :class="task.checked ? 'line-through' : ''"
                            class="text-xl">{{ task.title }}
                        </span>
                    </div>
                    <div class="flex space-x-2">
                        <span
                            class="text-end cursor-pointer hover:text-amber-500 duration-500"
                            @click="editTask(task)"
                        >
                            <PencilSquareIcon class="w-6 h-6"/>
                        </span>
                        <span
                            class="text-end cursor-pointer hover:text-amber-500 duration-500"
                            @click="removeTask(index)"
                        >
                            <XMarkIcon class="w-6 h-6"/>
                        </span>
                    </div>
                </div>
            </div>

            <div v-else-if="notAcceptedToDo.length && tabActive === false" class="col-span-1 space-y-2">
                <div
                    v-for="(task, index) in notAcceptedToDo" :key="task.id"
                    class="flex justify-between items-center px-6 py-2 bg-blue-500 rounded-2xl text-white shadow
                    space-x-2 bg-gradient-to-r from-cyan-500 to-blue-500"
                >
                    <div class="flex space-x-2 items-center">
                        <input
                            type="checkbox"
                            class="w-5 h-5 text-start text-green-600 bg-white cursor-pointer hover:bg-green-200
                                hover:text-green-700 duration-500 rounded"
                            v-if="task.checked"
                            v-model="task.checked"
                            @change="checkArray"
                        />
                        <input
                            type="checkbox"
                            class="w-5 h-5 text-start text-red-600 bg-white cursor-pointer hover:bg-red-200
                                hover:text-red-700 duration-500 rounded"
                            v-else
                            v-model="task.checked"
                            @change="checkArray"
                        />
                        <span class="text-xl">{{ index + 1 + '.' }}</span>
                        <span
                            :class="task.checked ? 'line-through' : ''"
                            class="text-xl">{{ task.title }}
                        </span>
                    </div>
                    <div class="flex space-x-2">
                        <span
                            class="text-end cursor-pointer hover:text-amber-500 duration-500"
                            @click="editTask(task)"
                        >
                            <PencilSquareIcon class="w-6 h-6"/>
                        </span>
                        <span
                            class="text-end cursor-pointer hover:text-amber-500 duration-500"
                            @click="removeTask(index)"
                        >
                            <XMarkIcon class="w-6 h-6"/>
                        </span>
                    </div>
                </div>
            </div>

            <div v-else class="text-center mt-12">
                <h1 class="font-semibold text-3xl text-fuchsia-700">There is no any tasks</h1>
            </div>

            <div v-if="toDoList.length" class="flex justify-between px-6">
                <div class="space-x-1 space-y-2 items-center">
                    <div @click="selectAll" class="flex items-center space-x-2">
                        <input
                            type="checkbox"
                            id="selectAll"
                            class="w-5 h-5 text-start text-green-600 bg-white cursor-pointer hover:bg-green-200
                            hover:text-green-700 duration-500 rounded border border-gray-500 cursor-pointer"
                            v-model="selectAllToDo"
                            :checked="toDoList.every(el => el.checked === true)"
                        />
                        <label
                            for="selectAll"
                            @click="selectAll"
                            class="font-semibold cursor-pointer text-lg text-blue-500 hover:text-blue-600 duration-500"
                        >
                            Select all
                        </label>
                    </div>
                    <div v-if="toDoList.some(el => el.checked === true)" class="flex space-x-2">
                        <TrashIcon class="w-6 h-6 -ml-2 cursor-pointer text-red-400 hover:text-red-600 duration-500"/>
                        <div>
                            <span @click="removeChecked"
                                  class="cursor-pointer font-semibold text-lg text-red-400 hover:text-red-600 duration-500"
                            >
                                Remove checked
                            </span>
                        </div>
                    </div>
                </div>

                <div class="flex space-x-1">
                    <div>
                        <span
                            class="font-semibold text-lg cursor-pointer text-red-400 hover:text-red-600 duration-500"
                            @click="removeAll"
                        >
                            Remove all
                        </span>
                    </div>
                    <TrashIcon
                        class="w-6 h-6 text-red-400 cursor-pointer hover:text-red-600 duration-500"
                        @click="removeAll"
                    />
                </div>
            </div>

            <div v-if="toDoList.length" class="col-span-1 space-y-1 px-6 items-center">
                <hr class="h-1 bg-blue-500 my-4">
                <div class="flex justify-between">
                    <div>
                        <div>
                            <span class="text-xl font-semibold text-gray-500">All tasks: </span>
                            <span class="text-xl font-semibold text-blue-500">{{ toDoList.length }}</span>
                        </div>
                        <div>
                            <span class="text-xl font-semibold text-gray-500">Accepted tasks: </span>
                            <span class="text-xl font-semibold text-blue-500">
                        {{ getDoneTasks }}
                    </span>
                        </div>
                    </div>
                    <div>
                        <div class="space-x-2">
                            <input @change="filterToDoList(allToDoList)" type="radio" id="radio1" name="tasks" checked
                                   class="cursor-pointer">
                            <label for="radio1" class="font-bold text-gray-500 cursor-pointer hover:text-blue-500
                            duration-500 text-lg cursor-pointer">All tasks</label>
                        </div>
                        <div class="space-x-2">
                            <input @change="filterToDoList(accepted)" type="radio" id="radio2" name="tasks"
                                   class="cursor-pointer">
                            <label for="radio2" class="font-bold text-gray-500 cursor-pointer hover:text-blue-500
                            duration-500 text-lg cursor-pointer">Accepted</label>
                        </div>
                        <div class="space-x-2">
                            <input @change="filterToDoList(notAccepted)" type="radio" id="radio3" name="tasks"
                                   class="cursor-pointer">
                            <label for="radio3" class="font-bold text-gray-500 cursor-pointer hover:text-blue-500
                            duration-500 text-lg cursor-pointer">Not
                                Accepted</label>
                        </div>
                    </div>
                </div>
            </div>

        </div>
    </div>

</template>

<script>
import {
    CheckIcon,
    MagnifyingGlassIcon,
    MinusIcon,
    PencilSquareIcon,
    TrashIcon,
    XMarkIcon
} from '@heroicons/vue/24/outline'

export default {
    components: {
        PencilSquareIcon,
        TrashIcon,
        CheckIcon,
        MinusIcon,
        XMarkIcon,
        MagnifyingGlassIcon
    },
    data() {
        return {
            theme: 'Todo App',
            title: 'todo',
            idCreater: 0,
            value: '',
            search: '',
            currentToDoId: null,
            isEdit: false,
            checked: false,
            toDoList: [],
            selectAllToDo: false,

            accepted: true,
            notAccepted: false,
            allToDoList: null,
            acceptedToDo: [],
            notAcceptedToDo: [],
            tabActive: null,

            searchListToDo: []
        }
    },
    computed: {
        getDoneTasks() {
            return this.toDoList.filter(el => el.checked === true).length
        }
    },
    methods: {
        addTask() {
            if (this.value.length) {
                this.toDoList.push({
                    id: this.idCreater++,
                    title: this.value[0].toUpperCase() + this.value.slice(1),
                    checked: this.checked
                })
                this.value = ''
            }
        },
        editTask(item) {
            this.value = item.title
            this.currentToDoId = item.id
        },
        updateTask() {
            if (this.value.length) {
                const index = this.toDoList.findIndex(el => el.id === this.currentToDoId)
                this.toDoList[index].title = this.value
                this.currentToDoId = this.value = null
            }
        },
        addOrUpdate() {
            if (this.currentToDoId === null) {
                this.addTask()
            } else {
                this.updateTask()
            }
        },
        removeTask(index) {
            this.toDoList.splice(index, 1)
            this.value = ''
        },
        removeAll() {
            this.toDoList = []
            this.value = ''
        },
        selectAll() {
            this.selectAllToDo = !this.selectAllToDo
            if (this.selectAllToDo) {
                this.toDoList.map(el => el.checked = true)
            } else {
                this.toDoList.map(el => el.checked = false)
            }
        },
        removeChecked() {
            this.toDoList = this.toDoList.filter(el => el.checked === false)
        },
        checkArray() {
            this.filterToDoList(this.tabActive)
        },
        filterToDoList(item) {
            this.tabActive = item
            if (item === true) {
                this.acceptedToDo = this.toDoList.filter(el => el.checked === true)
            } else if (item === false) {
                this.notAcceptedToDo = this.toDoList.filter(el => el.checked === false)
            }
        },
        searchList() {
            this.searchListToDo = this.toDoList.filter(el => el.title.toLowerCase().startsWith(this.search))
        }
    },
    mounted() {
        this.idCreater = this.toDoList.length + 1
    }
}
</script>

<style scoped>
</style>
