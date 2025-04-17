<script setup>
import { computed } from 'vue';

const props = defineProps({
  weatherData: {
    type: Object,
    required: true
  }
});

const temperature = computed(() => Math.round(props.weatherData.main.temp));
const feelsLike = computed(() => Math.round(props.weatherData.main.feels_like));
const weatherDescription = computed(() => 
  props.weatherData.weather[0].description.charAt(0).toUpperCase() + 
  props.weatherData.weather[0].description.slice(1)
);
const humidity = computed(() => props.weatherData.main.humidity);
const windSpeed = computed(() => Math.round(props.weatherData.wind.speed * 3.6)); // Convert m/s to km/h
const iconCode = computed(() => props.weatherData.weather[0].icon);
const iconUrl = computed(() => `https://openweathermap.org/img/wn/${iconCode.value}@2x.png`);
const date = computed(() => {
  const now = new Date();
  return now.toLocaleDateString('fr-FR', { 
    weekday: 'long', 
    year: 'numeric', 
    month: 'long', 
    day: 'numeric' 
  });
});
const time = computed(() => {
  const now = new Date();
  return now.toLocaleTimeString('fr-FR', { 
    hour: '2-digit', 
    minute: '2-digit' 
  });
});
</script>

<template>
  <div class="weather-display">
    <div class="weather-header">
      <h2>{{ weatherData.name }}, {{ weatherData.sys.country }}</h2>
      <p class="date-time">{{ date }}, {{ time }}</p>
    </div>
    
    <div class="weather-main">
      <div class="temp-container">
        <div class="temperature">{{ temperature }}°C</div>
        <div class="feels-like">Ressenti: {{ feelsLike }}°C</div>
      </div>
      
      <div class="weather-icon">
        <img :src="iconUrl" :alt="weatherDescription" />
        <div class="description">{{ weatherDescription }}</div>
      </div>
    </div>
    
    <div class="weather-details">
      <div class="detail">
        <div class="detail-icon">💧</div>
        <div class="detail-info">
          <div class="detail-label">Humidité</div>
          <div class="detail-value">{{ humidity }}%</div>
        </div>
      </div>
      
      <div class="detail">
        <div class="detail-icon">💨</div>
        <div class="detail-info">
          <div class="detail-label">Vent</div>
          <div class="detail-value">{{ windSpeed }} km/h</div>
        </div>
      </div>
      
      <div class="detail">
        <div class="detail-icon">🌡️</div>
        <div class="detail-info">
          <div class="detail-label">Pression</div>
          <div class="detail-value">{{ weatherData.main.pressure }} hPa</div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.weather-display {
  display: flex;
  flex-direction: column;
  gap: 20px;
  padding: 20px;
  border-radius: 10px;
  background-color: white;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

.weather-header {
  text-align: center;
  margin-bottom: 10px;
}

.weather-header h2 {
  margin: 0;
  font-size: 1.8rem;
  color: #333;
}

.date-time {
  color: #666;
  margin: 5px 0 0;
  font-size: 1rem;
}

.weather-main {
  display: flex;
  justify-content: space-around;
  align-items: center;
  padding: 10px 0;
}

.temperature {
  font-size: 3.5rem;
  font-weight: bold;
  color: #333;
}

.feels-like {
  font-size: 1rem;
  color: #666;
}

.weather-icon {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.weather-icon img {
  width: 100px;
  height: 100px;
}

.description {
  text-align: center;
  font-size: 1.2rem;
  margin-top: 5px;
  color: #444;
}

.weather-details {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  padding-top: 15px;
  border-top: 1px solid #eee;
}

.detail {
  display: flex;
  align-items: center;
  gap: 8px;
  flex: 1;
}

.detail-icon {
  font-size: 1.5rem;
}

.detail-info {
  display: flex;
  flex-direction: column;
}

.detail-label {
  font-size: 0.85rem;
  color: #666;
}

.detail-value {
  font-size: 1.1rem;
  font-weight: 500;
}

@media (max-width: 600px) {
  .weather-main {
    flex-direction: column;
    gap: 20px;
  }
  
  .weather-details {
    flex-direction: column;
    gap: 15px;
  }
}
</style> 