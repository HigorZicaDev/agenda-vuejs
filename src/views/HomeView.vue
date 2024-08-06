<script setup>
import { ref, reactive, onMounted } from "vue";
import PulseLoader from 'vue-spinner/src/PulseLoader.vue';
import axios from "axios";

    const isLoading = ref(true);
    const state = reactive({
        events: [],
    });

    const hoje = new Date()
    const dia = hoje.getDate().toString().padStart(2,'0')
    const mes = String(hoje.getMonth() + 1).padStart(2,'0')
    const ano = hoje.getFullYear()
    const dataAtual = `${dia} / ${mes} / ${ano}`


    onMounted(async () => {
        try {
            const response = await axios.get('https://brasileirosemmontpellier.blog/api/events');
            let data = response.data;
            
            // console.log(data);
            const today = new Date().toISOString().split('T')[0];

            // Filtrar os eventos cujo time.start começa hoje
            const eventsToday = data.filter(event => {
              return event.start.startsWith(today);
            });

            // console.log(eventsToday);

            // Ordenar os eventos pelo horário de início (start)
            eventsToday.sort((a, b) => {
              return new Date(a.start) - new Date(b.end);
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
    <h3 class="text-3xl">Aulas de Hoje: {{ dataAtual }}</h3>
        <br>
          <div v-if="state.events.length < 1">
            Não tem agendamentos para a data de Hoje!
          </div>
          <ul v-else class="border border-gray-200 rounded overflow-hidden shadow-md">
            <li v-for="event in state.events" :key="event.id" 
            class="px-4 py-2 bg-white hover:bg-sky-100 hover:text-sky-900 border-b last:border-none border-gray-200 transition-all duration-300 ease-in-out">
              <span class="text-xl font-medium">{{ event.title }}</span> <br>
              <span>{{ event.with }}</span> <br>
              <span>Inicio: {{ event.start.slice(10, 16) }}</span> - <span>Final: {{ event.end.slice(10, 16) }}</span>

            </li>

          </ul>
  </div>
</template>



<style>

  .container {
    padding: 30px;
  }
</style>