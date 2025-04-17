<script setup>
import { computed } from 'vue';

const props = defineProps({
  forecastData: {
    type: Object,
    required: true
  }
});

const dailyForecast = computed(() => {
  const forecasts = props.forecastData.list;
  const dailyData = {};
  
  // Group forecast data by day
  forecasts.forEach(item => {
    const date = new Date(item.dt * 1000);
    const day = date.toLocaleDateString('fr-FR', { weekday: 'long' });
    
    if (!dailyData[day]) {
      dailyData[day] = {
        temps: [],
        icons: [],
        descriptions: []
      };
    }
    
    dailyData[day].temps.push(item.main.temp);
    dailyData[day].icons.push(item.weather[0].icon);
    dailyData[day].descriptions.push(item.weather[0].description);
  });
  
  // Process the grouped data to get daily summary
  return Object.keys(dailyData).map(day => {
    const dayData = dailyData[day];
    const avgTemp = dayData.temps.reduce((sum, temp) => sum + temp, 0) / dayData.temps.length;
    
    // Get the most frequent icon and description
    const iconCount = {};
    dayData.icons.forEach(icon => {
      iconCount[icon] = (iconCount[icon] || 0) + 1;
    });
    const mostFrequentIcon = Object.keys(iconCount).reduce((a, b) => 
      iconCount[a] > iconCount[b] ? a : b
    );
    
    const descriptionCount = {};
    dayData.descriptions.forEach(desc => {
      descriptionCount[desc] = (descriptionCount[desc] || 0) + 1;
    });
    const mostFrequentDescription = Object.keys(descriptionCount).reduce((a, b) => 
      descriptionCount[a] > descriptionCount[b] ? a : b
    );
    
    return {
      day,
      temp: Math.round(avgTemp),
      icon: mostFrequentIcon,
      description: mostFrequentDescription,
      minTemp: Math.round(Math.min(...dayData.temps)),
      maxTemp: Math.round(Math.max(...dayData.temps))
    };
  }).slice(0, 5); // Limit to 5 days
});
</script>

<template>
  <div class="daily-forecast">
    <h3>Prévisions à 5 jours</h3>
    
    <div class="daily-container">
      <div 
        v-for="(forecast, index) in dailyForecast" 
        :key="index" 
        class="daily-item"
      >
        <div class="daily-day">{{ forecast.day }}</div>
        <div class="daily-icon-container">
          <img 
            :src="`https://openweathermap.org/img/wn/${forecast.icon}@2x.png`" 
            :alt="forecast.description" 
            class="daily-icon"
          />
          <div class="daily-description">{{ forecast.description }}</div>
        </div>
        <div class="daily-temps">
          <span class="daily-temp-min">{{ forecast.minTemp }}°</span>
          <span class="daily-temp-max">{{ forecast.maxTemp }}°</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.daily-forecast {
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

.daily-container {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.daily-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 15px;
  border-radius: 8px;
  background-color: rgba(110, 142, 251, 0.05);
  transition: transform 0.2s ease;
}

.daily-item:hover {
  transform: translateX(3px);
  background-color: rgba(110, 142, 251, 0.1);
}

.daily-day {
  font-weight: 500;
  font-size: 1.1rem;
  min-width: 100px;
  text-transform: capitalize;
}

.daily-icon-container {
  display: flex;
  align-items: center;
  flex: 1;
}

.daily-icon {
  width: 50px;
  height: 50px;
}

.daily-description {
  margin-left: 10px;
  color: #555;
  text-transform: capitalize;
}

.daily-temps {
  display: flex;
  gap: 15px;
  font-weight: 500;
}

.daily-temp-min {
  color: #666;
}

.daily-temp-max {
  color: #333;
}

@media (max-width: 600px) {
  .daily-item {
    flex-direction: column;
    text-align: center;
    gap: 10px;
  }
  
  .daily-icon-container {
    flex-direction: column;
    margin: 10px 0;
  }
  
  .daily-description {
    margin-left: 0;
    margin-top: 5px;
  }
}
</style> 