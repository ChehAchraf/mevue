<script setup>
import { computed } from 'vue';

const props = defineProps({
  forecastData: {
    type: Object,
    required: true
  }
});

const hourlyForecast = computed(() => {
  // Get the next 12 hours (8 records with 3-hour intervals)
  return props.forecastData.list.slice(0, 8).map(item => {
    const date = new Date(item.dt * 1000);
    return {
      time: date.toLocaleTimeString('fr-FR', { hour: '2-digit', minute: '2-digit' }),
      day: date.toLocaleDateString('fr-FR', { weekday: 'short' }),
      temp: Math.round(item.main.temp),
      icon: item.weather[0].icon,
      description: item.weather[0].description
    };
  });
});
</script>

<template>
  <div class="hourly-forecast">
    <h3>Prévisions à court terme</h3>
    
    <div class="forecast-container">
      <div 
        v-for="(forecast, index) in hourlyForecast" 
        :key="index" 
        class="forecast-item"
      >
        <div class="forecast-time">{{ forecast.day }} {{ forecast.time }}</div>
        <img 
          :src="`https://openweathermap.org/img/wn/${forecast.icon}.png`" 
          :alt="forecast.description" 
          class="forecast-icon"
        />
        <div class="forecast-temp">{{ forecast.temp }}°C</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.hourly-forecast {
  margin-top: 20px;
  padding: 20px;
  border-radius: 10px;
  background-color: white;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

h3 {
  margin-top: 0;
  margin-bottom: 20px;
  font-size: 1.5rem;
  color: #333;
  text-align: center;
}

.forecast-container {
  display: flex;
  overflow-x: auto;
  gap: 15px;
  padding: 10px 5px;
  scrollbar-width: thin;
  scrollbar-color: #ddd transparent;
}

.forecast-container::-webkit-scrollbar {
  height: 6px;
}

.forecast-container::-webkit-scrollbar-track {
  background: transparent;
}

.forecast-container::-webkit-scrollbar-thumb {
  background-color: #ddd;
  border-radius: 10px;
}

.forecast-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 100px;
  padding: 10px;
  border-radius: 8px;
  background-color: rgba(110, 142, 251, 0.05);
  transition: transform 0.2s ease;
}

.forecast-item:hover {
  transform: translateY(-3px);
  background-color: rgba(110, 142, 251, 0.1);
}

.forecast-time {
  font-size: 0.85rem;
  color: #666;
  text-align: center;
}

.forecast-icon {
  margin: 5px 0;
  width: 50px;
  height: 50px;
}

.forecast-temp {
  font-size: 1.2rem;
  font-weight: 500;
  color: #333;
}

@media (max-width: 600px) {
  .forecast-item {
    min-width: 80px;
  }
  
  .forecast-time {
    font-size: 0.75rem;
  }
  
  .forecast-temp {
    font-size: 1rem;
  }
}
</style> 