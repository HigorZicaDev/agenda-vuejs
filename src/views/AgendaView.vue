<script setup>
import axios from "axios";
import { Qalendar } from "qalendar";
import { onMounted, reactive, ref } from "vue";
import PulseLoader from 'vue-spinner/src/PulseLoader.vue';
import router from '@/router';
import { useToast } from "vue-toastification";

        const events = ref([]);
        const config = ref({
                "dayBoundaries": {
                    "start": 6,
                    "end": 21
                }
        });
        const loading = ref(true);
        const showModal = ref(false);

        const form = reactive({
            student: '',
            teacher: '',
            start: '',
            end: '',
            color: '',
            description: '',
            isEditable: 1,
        });

        const toast = useToast();

        const saveEventClick = async () => {
            const newEvent = {
                title: form.student,
                with: form.teacher,
                start: form.start.replace("T", " "),
                end: form.end.replace("T", " "),
                color: form.color,
                description: form.description,
                isEditable: 1,
            };

            // console.log(newEvent);

            try {
                const request = await axios.post('http://apiagenda.test/api/events', newEvent);
                // console.log(request);
                getAllEvents();
                router.push({ name: 'agenda' });
                showModal.value = false;
                toast.success('Event Added Successfully');
            } catch (error) {
                console.log('Error fetching job', error);
                toast.error('Event Was Not Added');
            }
        };

        const handleEventClick = (event) => {
            // console.log(event.clickedEvent.id);
            // let eventId = event.clickedEvent.id;
        };

        
        const deleteEventClick = async (event) => {
            let eventId = event;
            try {
                await axios.delete(`http://apiagenda.test/api/events/${eventId}`)
                getAllEvents();
                router.push({ name: 'agenda' });
                showModal.value = false;
                toast.success('Event Deleted Successfully');
            } catch (error) {
                console.log('Error fetching job', error);
                toast.error('Event Was Not Deleted');
            }
        };
        
        const editEventClick = async (event) => {
            toast.warning('Não é possivel editar o evento no momento tente mais tarde!');
        };

        const handleNewEventClick = (event) => {
            console.log(event);
            showModal.value = true;
            form.start = event;
            form.end = event;
        };

        const closeModalClick = (event) => {
            showModal.value = !showModal.value;
        };

    onMounted(async () => {
        events.value = getAllEvents();
    });

    const getAllEvents = async (event) => {
        try {
            const response = await axios.get("http://apiagenda.test/api/events");
            let data = response.data;
            const formattedJson = {
                "events": data.map(event => ({
                    "id": event.id,
                    "title": event.title,
                    "with": event.with,
                    "time": {
                    "start": event.start,
                    "end": event.end
                    },
                    "color": event.color,
                    "description": event.description,
                    "isEditable": event.isEditable
                }))
            };
            events.value = formattedJson.events;

            loading.value = false;
        } catch (error) {
            console.error('Error: ', error);
        }
    }

</script>

<template>
  <div v-if="showModal">
    <div class="relative z-10" aria-labelledby="modal-title" role="dialog" aria-modal="true">

        <div class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity" aria-hidden="true"></div>

        <div class="fixed inset-0 z-10 w-screen">
            <div class="flex min-h-full items-end justify-center p-4 text-center sm:items-center sm:p-0">
            <form @submit.prevent="saveEventClick">
                <div class="relative rounded-lg bg-white text-left shadow-xl transition-all sm:my-8 sm:max-w-xl h-full w-[600px]">
                    <div class="bg-white px-4 pb-4 pt-5 sm:p-6 sm:pb-4">
                    <div class="sm:flex sm:items-start">
    
                        <div class="mt-3 text-center sm:ml-4 sm:mt-0 sm:text-left">
                            <h3 class="text-base font-semibold leading-6 text-gray-900 mb-3" id="modal-title">Cadastrar Aulas:</h3>
                            <div class="mt-2">
                                <div class="w-96 mt-3">
                                    <div>
                                        <label for="first_name" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Nome Aluno:</label>
                                        <input type="text" id="student" name="student" v-model="form.student" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="John" required />
                                    </div>
                                </div> 
                            </div>
                            <div class="mt-2">
                                <div class="w-96 mt-3">
                                    <div>
                                        <label for="first_name" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Nome Professor:</label>
                                        <input type="text" id="teacher" name="teacher" v-model="form.teacher" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="John" required />
                                    </div>
                                </div> 
                            </div>
                            <div class="mt-2">
                                <div class="w-96 mt-3">
                                    <div>
                                        <label for="message" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Observações sobre a Aula:</label>
                                        <textarea v-model="form.description" id="description" name="description" rows="4" class="block p-2.5 w-full text-sm text-gray-900 bg-gray-50 rounded-lg border border-gray-300 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="Write your thoughts here..."></textarea>
                                    </div>
                                </div> 
                            </div>
                            <div class="mt-2">
                                <div class="flex w-96 mt-3 gap-4">
                                    <div class="flex flex-col">
                                        <div class="flex flex-wrap">
                                            <div class="flex items-center me-4">
                                                <input id="red-radio" type="radio" value="red" v-model="form.color" name="colored-radio" class="w-4 h-4 text-red-600 bg-gray-100 border-gray-300 focus:ring-red-500 dark:focus:ring-red-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600">
                                                <label for="red-radio" class="ms-2 text-sm font-medium text-gray-900 dark:text-gray-300">Vermelho</label>
                                            </div>
                                            <div class="flex items-center me-4">
                                                <input id="green-radio" type="radio" value="green" v-model="form.color" name="colored-radio" class="w-4 h-4 text-green-600 bg-gray-100 border-gray-300 focus:ring-green-500 dark:focus:ring-green-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600">
                                                <label for="green-radio" class="ms-2 text-sm font-medium text-gray-900 dark:text-gray-300">Verde</label>
                                            </div>
                                            <div class="flex items-center me-4">
                                                <input checked id="purple-radio" type="radio" value="blue" v-model="form.color" name="colored-radio" class="w-4 h-4 text-purple-600 bg-gray-100 border-gray-300 focus:ring-purple-500 dark:focus:ring-purple-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600">
                                                <label for="purple-radio" class="ms-2 text-sm font-medium text-gray-900 dark:text-gray-300">Azul</label>
                                            </div>
                                        </div>
                                    </div>
                                </div> 
                            </div>
                            <div class="mt-2">
                                <div class="flex w-96 mt-3 gap-4">
                                    <div class="flex flex-col">
                                        <label for="message" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Inicio:</label>
                                        <input type="datetime-local" v-model="form.start" id="start" name="start" class="block p-2.5 w-full text-sm text-gray-900 bg-gray-50 rounded-lg border border-gray-300 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="00:00"></input>
                                    </div>
                                    <div class="flex flex-col">
                                        <label for="message" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Final:</label>
                                        <input type="datetime-local" v-model="form.end" id="end" name="end" class="block p-2.5 w-full text-sm text-gray-900 bg-gray-50 rounded-lg border border-gray-300 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="00:00"></input>
                                    </div>
                                </div> 
                            </div>
                        </div>
                    </div>
                    </div>
                    <div class="bg-gray-50 px-4 py-3 sm:flex sm:flex-row-reverse sm:px-6">
                    <button type="submit" class="inline-flex w-full justify-center rounded-md bg-green-600 px-3 py-2 text-sm font-semibold text-white shadow-sm hover:bg-green-500 sm:ml-3 sm:w-auto">Salvar</button>
                    <button @click="closeModalClick()" type="button" class="mt-3 inline-flex w-full justify-center rounded-md bg-white px-3 py-2 text-sm font-semibold text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 hover:bg-gray-50 sm:mt-0 sm:w-auto">Cancel</button>
                    </div>
                </div>
            </form>    
            </div>
        </div>
        </div>
  </div>
  <div v-if="loading === true" class="loading-effect">
    <PulseLoader class="loading-effect"></PulseLoader>
  </div>
  <div v-else class="container">
    <Qalendar 
      :events="events"
      :config="config"
      @event-was-clicked="handleEventClick"
      @datetime-was-clicked="handleNewEventClick"
      @delete-event="deleteEventClick"
      @edit-event="editEventClick"
    />
  </div>
</template>



<style>
  @import "qalendar/dist/style.css";

  .loading-effect{
    text-align: center;
    color: gray;
    padding: 30px;
  }
</style>/