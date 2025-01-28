<script setup lang="ts">
import { ref, computed } from 'vue'
import { format, addDays, startOfWeek, addWeeks, subWeeks, parse, isWithinInterval, isSameDay } from 'date-fns'

interface Event {
  id: number
  title: string
  start: string
  end: string
  color: string
  date: string
  description?: string
  location?: string
  isRecurring?: boolean
}

const currentDate = ref(new Date())
const weekDays = ['MON', 'TUE', 'WED', 'THU', 'FRI', 'SAT']
const showEventModal = ref(false)
const showEventDetails = ref(false)
const draggedEvent = ref<Event | null>(null)
const selectedTimeSlot = ref({ hour: 0, day: 0 })
const selectedEvent = ref<Event | null>(null)
const showConfirmDelete = ref(false)

const newEvent = ref({
  title: '',
  description: '',
  location: '',
  start: '09:00',
  end: '10:00',
  color: 'bg-blue-100',
  isRecurring: false
})

const events = ref<Event[]>([
  {
    id: 1,
    title: 'Marketing Sync',
    description: 'Weekly marketing team sync meeting',
    location: 'Conference Room A',
    start: '11:00',
    end: '12:00',
    color: 'bg-red-100',
    date: '2024-03-17',
    isRecurring: true
  },
  {
    id: 2,
    title: 'Lunch',
    description: 'Team lunch',
    location: 'Cafeteria',
    start: '12:00',
    end: '13:00',
    color: 'bg-green-100',
    date: '2024-03-17'
  },
  {
    id: 3,
    title: 'Doctor appointment',
    description: 'Annual checkup',
    location: 'Medical Center',
    start: '10:30',
    end: '11:15',
    color: 'bg-blue-100',
    date: '2024-03-19'
  }
])

const colorOptions = [
  { name: 'Blue', class: 'bg-blue-100', border: 'border-blue-300' },
  { name: 'Red', class: 'bg-red-100', border: 'border-red-300' },
  { name: 'Green', class: 'bg-green-100', border: 'border-green-300' },
  { name: 'Yellow', class: 'bg-yellow-100', border: 'border-yellow-300' },
  { name: 'Purple', class: 'bg-purple-100', border: 'border-purple-300' },
  { name: 'Pink', class: 'bg-pink-100', border: 'border-pink-300' },
  { name: 'Indigo', class: 'bg-indigo-100', border: 'border-indigo-300' }
]

const currentWeekDates = computed(() => {
  const start = startOfWeek(currentDate.value, { weekStartsOn: 1 })
  return weekDays.map((_, index) => addDays(start, index))
})

const formattedWeekRange = computed(() => {
  const start = currentWeekDates.value[0]
  const end = currentWeekDates.value[5]
  return `${format(start, 'MMMM d')} - ${format(end, 'd')}`
})

const isToday = (date: Date) => {
  return isSameDay(date, new Date())
}

const nextWeek = () => {
  currentDate.value = addWeeks(currentDate.value, 1)
}

const previousWeek = () => {
  currentDate.value = subWeeks(currentDate.value, 1)
}

const goToToday = () => {
  currentDate.value = new Date()
}

const openEventModal = (hour: number, day: number) => {
  selectedTimeSlot.value = { hour, day }
  newEvent.value = {
    title: '',
    description: '',
    location: '',
    start: `${hour.toString().padStart(2, '0')}:00`,
    end: `${(hour + 1).toString().padStart(2, '0')}:00`,
    color: 'bg-blue-100',
    isRecurring: false
  }
  showEventModal.value = true
}

const openEventDetails = (event: Event) => {
  selectedEvent.value = event
  showEventDetails.value = true
}

const createEvent = () => {
  const selectedDate = currentWeekDates.value[selectedTimeSlot.value.day]
  const dateStr = format(selectedDate, 'yyyy-MM-dd')
  
  const newEventData = {
    id: events.value.length + 1,
    ...newEvent.value,
    date: dateStr
  }
  
  events.value.push(newEventData)
  
  if (newEventData.isRecurring) {
    // Create recurring events for the next 4 weeks
    for (let i = 1; i <= 4; i++) {
      const recurringDate = addWeeks(selectedDate, i)
      events.value.push({
        ...newEventData,
        id: events.value.length + 1,
        date: format(recurringDate, 'yyyy-MM-dd')
      })
    }
  }
  
  showEventModal.value = false
}

const confirmDelete = (event: Event) => {
  selectedEvent.value = event
  showConfirmDelete.value = true
}

const deleteEvent = () => {
  if (!selectedEvent.value) return
  
  if (selectedEvent.value.isRecurring) {
    // Delete all recurring instances
    events.value = events.value.filter(e => 
      !(e.title === selectedEvent.value?.title && 
        e.start === selectedEvent.value?.start && 
        e.isRecurring)
    )
  } else {
    events.value = events.value.filter(e => e.id !== selectedEvent.value?.id)
  }
  
  showConfirmDelete.value = false
  showEventDetails.value = false
}

const startDrag = (event: Event) => {
  draggedEvent.value = event
}

const onDrop = (hour: number, day: number) => {
  if (!draggedEvent.value) return
  
  const selectedDate = currentWeekDates.value[day]
  const dateStr = format(selectedDate, 'yyyy-MM-dd')
  
  // Calculate duration of original event
  const [startHour, startMinute] = draggedEvent.value.start.split(':').map(Number)
  const [endHour, endMinute] = draggedEvent.value.end.split(':').map(Number)
  const durationHours = endHour - startHour + (endMinute - startMinute) / 60
  
  // Create new start and end times
  const newStart = `${hour.toString().padStart(2, '0')}:00`
  const newEnd = `${(hour + Math.floor(durationHours)).toString().padStart(2, '0')}:${((durationHours % 1) * 60).toString().padStart(2, '0')}`
  
  // Update the event
  const eventIndex = events.value.findIndex(e => e.id === draggedEvent.value?.id)
  if (eventIndex !== -1) {
    events.value[eventIndex] = {
      ...draggedEvent.value,
      start: newStart,
      end: newEnd,
      date: dateStr
    }
  }
  
  draggedEvent.value = null
}

const onDragOver = (e: DragEvent) => {
  e.preventDefault()
}

const getEventsForDay = (date: Date) => {
  const dateStr = format(date, 'yyyy-MM-dd')
  return events.value.filter(event => event.date === dateStr)
}

const formatTime = (time: string) => {
  const [hours, minutes] = time.split(':')
  return `${hours}:${minutes}`
}
</script>

<template>
  <div class="flex-1 p-6 bg-gray-50">
    <div class="bg-white rounded-xl shadow-lg p-6 border border-gray-100">
      <!-- Header Section -->
      <div class="flex items-center justify-between mb-8">
        <div class="flex items-center space-x-4">
          <button @click="previousWeek" 
                  class="btn btn-secondary p-2 rounded-full">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
            </svg>
          </button>
          <h2 class="text-2xl font-semibold text-gray-800">{{ formattedWeekRange }}</h2>
          <button @click="nextWeek" 
                  class="btn btn-secondary p-2 rounded-full">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
            </svg>
          </button>
          <button @click="goToToday" 
                  class="btn btn-primary ml-4">
            Today
          </button>
        </div>
        
        <div class="flex items-center space-x-4">
          <button class="btn btn-secondary">
            <svg class="w-5 h-5 mr-2 inline-block" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 4h13M3 8h9m-9 4h6m4 0l4-4m0 0l4 4m-4-4v12" />
            </svg>
            Filter
          </button>
          <button class="btn btn-secondary">
            <svg class="w-5 h-5 mr-2 inline-block" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6" />
            </svg>
            New Event
          </button>
        </div>
      </div>

      <!-- Days Header -->
      <div class="grid grid-cols-6 gap-4 mb-6">
        <div v-for="(date, index) in currentWeekDates" 
             :key="index" 
             class="text-center">
          <div class="text-sm font-medium text-gray-500 mb-2">{{ weekDays[index] }}</div>
          <div :class="[
            'text-2xl font-light rounded-full w-12 h-12 flex items-center justify-center mx-auto transition-all duration-200',
            isToday(date) ? 'bg-blue-500 text-white shadow-md' : 'hover:bg-gray-100'
          ]">
            {{ format(date, 'd') }}
          </div>
        </div>
      </div>

      <!-- Calendar Grid -->
      <div class="mt-8 relative calendar-grid">
        <!-- Time Labels -->
        <div class="absolute left-0 w-16">
          <div v-for="hour in 24" :key="hour" 
               class="h-16 relative -mt-2">
            <span class="text-sm font-medium text-gray-400">
              {{ (hour - 1).toString().padStart(2, '0') }}:00
            </span>
          </div>
        </div>

        <!-- Calendar Content -->
        <div class="ml-16">
          <div class="grid grid-cols-6 gap-4">
            <div v-for="(date, dayIndex) in currentWeekDates" 
                 :key="dayIndex" 
                 :class="['relative h-[960px]', isToday(date) ? 'today-column' : '']"
                 @dragover="onDragOver"
                 @drop="onDrop(Math.floor($event.offsetY / 64), dayIndex)">
              <!-- Time Slots -->
              <div class="absolute inset-0">
                <div v-for="hour in 24" :key="hour"
                     @click="openEventModal(hour - 1, dayIndex)"
                     class="h-16 relative group cursor-pointer">
                  <div class="time-grid-line"></div>
                  <div class="absolute inset-0 group-hover:bg-blue-50 opacity-0 group-hover:opacity-25 transition-opacity"></div>
                </div>
              </div>
              
              <!-- Events -->
              <div v-for="event in getEventsForDay(date)" 
                   :key="event.id" 
                   class="absolute w-[95%] p-3 rounded-lg shadow-sm cursor-move event-card"
                   :class="[event.color, event.color.replace('bg-', 'border-l-')]"
                   draggable="true"
                   @click.stop="openEventDetails(event)"
                   @dragstart="startDrag(event)"
                   :style="{
                     top: `${parseInt(event.start.split(':')[0]) * 64 + parseInt(event.start.split(':')[1]) / 60 * 64}px`,
                     height: `${(parseInt(event.end.split(':')[0]) - parseInt(event.start.split(':')[0])) * 64}px`
                   }">
                <div class="flex flex-col h-full">
                  <div class="flex items-center justify-between mb-1">
                    <div class="text-sm font-semibold truncate">{{ event.title }}</div>
                    <div v-if="event.isRecurring" 
                         class="text-xs text-gray-500">
                      <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
                              d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
                      </svg>
                    </div>
                  </div>
                  <div class="text-xs text-gray-600 font-medium">
                    {{ formatTime(event.start) }} — {{ formatTime(event.end) }}
                  </div>
                  <div v-if="event.location" 
                       class="text-xs text-gray-500 truncate mt-1 flex items-center">
                    <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
                            d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
                    </svg>
                    {{ event.location }}
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Event Creation Modal -->
    <Transition name="fade">
      <div v-if="showEventModal" 
           class="modal-backdrop">
        <div class="fixed inset-0 flex items-center justify-center p-4">
          <div class="modal-content p-6 max-w-lg w-full">
            <h3 class="text-xl font-semibold mb-6">Create New Event</h3>
            <div class="space-y-6">
              <div>
                <label class="block text-sm font-medium text-gray-700 mb-1">Title</label>
                <input v-model="newEvent.title" 
                       type="text" 
                       class="block w-full rounded-lg border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500">
              </div>
              
              <div>
                <label class="block text-sm font-medium text-gray-700 mb-1">Description</label>
                <textarea v-model="newEvent.description"
                          rows="3"
                          class="block w-full rounded-lg border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500"></textarea>
              </div>
              
              <div>
                <label class="block text-sm font-medium text-gray-700 mb-1">Location</label>
                <input v-model="newEvent.location"
                       type="text"
                       class="block w-full rounded-lg border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500">
              </div>

              <div class="grid grid-cols-2 gap-6">
                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-1">Start Time</label>
                  <input v-model="newEvent.start" 
                         type="time" 
                         class="block w-full rounded-lg border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500">
                </div>
                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-1">End Time</label>
                  <input v-model="newEvent.end" 
                         type="time" 
                         class="block w-full rounded-lg border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500">
                </div>
              </div>

              <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">Color</label>
                <div class="flex flex-wrap gap-3">
                  <button v-for="color in colorOptions" 
                          :key="color.class"
                          @click="newEvent.color = color.class"
                          :class="[
                            color.class,
                            'w-10 h-10 rounded-full border-2 transition-all duration-200 hover:scale-110',
                            color.border,
                            newEvent.color === color.class ? 'ring-2 ring-offset-2 ring-blue-500 scale-110' : ''
                          ]">
                  </button>
                </div>
              </div>

              <div class="flex items-center">
                <input type="checkbox"
                       v-model="newEvent.isRecurring"
                       class="rounded border-gray-300 text-blue-600 focus:ring-blue-500">
                <label class="ml-2 text-sm text-gray-700">Repeat weekly</label>
              </div>
            </div>
            
            <div class="mt-8 flex justify-end space-x-4">
              <button @click="showEventModal = false" 
                      class="btn btn-secondary">
                Cancel
              </button>
              <button @click="createEvent" 
                      class="btn btn-primary">
                Create Event
              </button>
            </div>
          </div>
        </div>
      </div>
    </Transition>

    <!-- Event Details Modal -->
    <Transition name="fade">
      <div v-if="showEventDetails && selectedEvent" 
           class="modal-backdrop">
        <div class="fixed inset-0 flex items-center justify-center p-4">
          <div class="modal-content p-6">
            <div class="flex justify-between items-start mb-6">
              <h3 class="text-xl font-semibold">{{ selectedEvent.title }}</h3>
              <button @click="showEventDetails = false" 
                      class="text-gray-400 hover:text-gray-600 text-2xl font-light">
                ×
              </button>
            </div>
            
            <div class="space-y-6">
              <div class="flex items-center text-gray-600">
                <svg class="w-5 h-5 mr-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
                        d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
                <span class="font-medium">{{ selectedEvent.start }} — {{ selectedEvent.end }}</span>
              </div>

              <div v-if="selectedEvent.location" class="flex items-center text-gray-600">
                <svg class="w-5 h-5 mr-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
                        d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
                </svg>
                <span>{{ selectedEvent.location }}</span>
              </div>

              <div v-if="selectedEvent.description" class="mt-4 p-4 bg-gray-50 rounded-lg">
                <p class="text-gray-600">{{ selectedEvent.description }}</p>
              </div>

              <div v-if="selectedEvent.isRecurring" class="flex items-center text-gray-600">
                <svg class="w-5 h-5 mr-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
                        d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
                </svg>
                <span>Repeats weekly</span>
              </div>
            </div>

            <div class="mt-8 flex justify-end space-x-4">
              <button @click="confirmDelete(selectedEvent)" 
                      class="btn btn-danger">
                Delete
              </button>
              <button @click="showEventDetails = false" 
                      class="btn btn-secondary">
                Close
              </button>
            </div>
          </div>
        </div>
      </div>
    </Transition>

    <!-- Delete Confirmation Modal -->
    <Transition name="fade">
      <div v-if="showConfirmDelete" 
           class="modal-backdrop">
        <div class="fixed inset-0 flex items-center justify-center p-4">
          <div class="modal-content p-6">
            <h3 class="text-xl font-semibold mb-4">Delete Event</h3>
            <p class="text-gray-600 mb-6">
              Are you sure you want to delete this event?
              <span v-if="selectedEvent?.isRecurring" class="block mt-2 text-sm bg-yellow-50 p-3 rounded-lg">
                ⚠️ This will delete all recurring instances of this event.
              </span>
            </p>
            
            <div class="flex justify-end space-x-4">
              <button @click="showConfirmDelete = false" 
                      class="btn btn-secondary">
                Cancel
              </button>
              <button @click="deleteEvent" 
                      class="btn btn-danger">
                Delete Event
              </button>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>