<script setup>
import { ref, reactive, onMounted } from "vue";
import PulseLoader from 'vue-spinner/src/PulseLoader.vue';
import axios from "axios";

    const isLoading = ref(true);
    const state = reactive({
        events: [],
    });


    onMounted(async () => {
        try {
            const response = await axios.get('http://localhost:5000/events');
            let data = response.data;
            
            // console.log(state.events);
            const today = new Date().toISOString().split('T')[0];

            // Filtrar os eventos cujo time.start começa hoje
            const eventsToday = data.filter(event => {
              return event.time.start.startsWith(today);
            });

            // console.log(eventsToday);

            // Ordenar os eventos pelo horário de início (time.start)
            eventsToday.sort((a, b) => {
              return new Date(a.time.start) - new Date(b.time.end);
            });

            state.events = eventsToday;


        } catch (error) {
            console.error('Error: ', error);
        } finally {
            isLoading.value = false;
        }
    });


</script>

<template>
  <div v-if="isLoading" class="loading-effect">
    <PulseLoader class="loading-effect"></PulseLoader>
  </div>
  <div v-else  class="container">
    <h3 class="text-3xl">Aulas de Hoje: 05/08/2024</h3>
        <br>
          <div v-if="state.events.length < 1">
            Não tem agendamentos para a data de Hoje!
          </div>
          <ul v-else class="border border-gray-200 rounded overflow-hidden shadow-md">
            <li v-for="event in state.events" :key="event.id" 
            class="px-4 py-2 bg-white hover:bg-sky-100 hover:text-sky-900 border-b last:border-none border-gray-200 transition-all duration-300 ease-in-out">
              <span class="text-xl font-medium">{{ event.title }}</span> <br>
              <span>{{ event.with }}</span> <br>
              <span>Inicio: {{ event.time.start.slice(10, 16) }}</span> - <span>Final: {{ event.time.end.slice(10, 16) }}</span>

            </li>

          </ul>
  </div>
</template>



<style>

  .container {
    padding: 30px;
  }
</style>