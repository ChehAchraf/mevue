<script setup>
import { ref, watch } from 'vue';
import axios from 'axios';

const emit = defineEmits(['search', 'geolocation']);
const searchQuery = ref('');
const geoLoading = ref(false);
const geoError = ref(null);
const suggestions = ref([]);
const isLoading = ref(false);

const handleSubmit = (e) => {
  e.preventDefault();
  if (searchQuery.value.trim()) {
    emit('search', searchQuery.value.trim());
    suggestions.value = [];
  }
};

const fetchSuggestions = async (query) => {
  if (query.length < 2) {
    suggestions.value = [];
    return;
  }
  
  isLoading.value = true;
  try {
    // Using OpenWeatherMap Geo API to get city suggestions
    const response = await axios.get(
      `https://api.openweathermap.org/geo/1.0/direct?q=${query}&limit=5&appid=1635890035cbba097fd5c26c8ea672a1`
    );
    
    suggestions.value = response.data.map(city => ({
      name: city.name,
      country: city.country,
      state: city.state,
      lat: city.lat,
      lon: city.lon
    }));
  } catch (error) {
    console.error('Error fetching city suggestions:', error);
    suggestions.value = [];
  } finally {
    isLoading.value = false;
  }
};

const selectSuggestion = (suggestion) => {
  searchQuery.value = suggestion.name;
  emit('geolocation', { lat: suggestion.lat, lon: suggestion.lon });
  suggestions.value = [];
};

// Debounce function to limit API calls while typing
let debounceTimeout;
watch(searchQuery, (newValue) => {
  clearTimeout(debounceTimeout);
  debounceTimeout = setTimeout(() => {
    if (newValue.trim()) {
      fetchSuggestions(newValue.trim());
    } else {
      suggestions.value = [];
    }
  }, 300);
});

const handleGeolocation = () => {
  if (!navigator.geolocation) {
    geoError.value = "La géolocalisation n'est pas prise en charge par votre navigateur";
    return;
  }

  geoLoading.value = true;
  geoError.value = null;

  navigator.geolocation.getCurrentPosition(
    (position) => {
      geoLoading.value = false;
      const { latitude, longitude } = position.coords;
      emit('geolocation', { lat: latitude, lon: longitude });
    },
    (error) => {
      geoLoading.value = false;
      
      switch (error.code) {
        case error.PERMISSION_DENIED:
          geoError.value = "Vous avez refusé la demande de géolocalisation";
          break;
        case error.POSITION_UNAVAILABLE:
          geoError.value = "Les informations de localisation ne sont pas disponibles";
          break;
        case error.TIMEOUT:
          geoError.value = "La demande de localisation a expiré";
          break;
        default:
          geoError.value = "Une erreur inconnue s'est produite";
          break;
      }
    }
  );
};
</script>

<template>
  <div class="search-wrapper">
    <div class="search-container-wrapper">
      <form @submit="handleSubmit" class="search-form">
        <div class="search-container">
          <input
            type="text"
            v-model="searchQuery"
            placeholder="Rechercher une ville..."
            class="search-input"
            autocomplete="off"
          />
          <button type="submit" class="search-button">
            <span class="search-icon">🔍</span>
          </button>
        </div>
      </form>
      
      <div class="suggestions-container" v-if="suggestions.length > 0">
        <ul class="suggestions-list">
          <li 
            v-for="(suggestion, index) in suggestions" 
            :key="index" 
            @click="selectSuggestion(suggestion)"
            class="suggestion-item"
          >
            <span class="suggestion-name">{{ suggestion.name }}</span>
            <span class="suggestion-details">
              {{ suggestion.state ? suggestion.state + ', ' : '' }}{{ suggestion.country }}
            </span>
          </li>
        </ul>
      </div>
      
      <div v-if="isLoading" class="suggestion-loading">Recherche de villes...</div>
    </div>
    
    <div class="geo-container">
      <button @click="handleGeolocation" class="geo-button" :disabled="geoLoading">
        <span v-if="geoLoading">Localisation...</span>
        <span v-else>📍 Ma position</span>
      </button>
      
      <div v-if="geoError" class="geo-error">{{ geoError }}</div>
    </div>
  </div>
</template>

<style scoped>
.search-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15px;
  width: 100%;
}

.search-container-wrapper {
  position: relative;
  width: 100%;
  max-width: 500px;
}

.search-form {
  width: 100%;
}

.search-container {
  display: flex;
  position: relative;
}

.search-input {
  flex: 1;
  padding: 12px 20px;
  font-size: 1rem;
  border: none;
  border-radius: 50px;
  background-color: rgba(255, 255, 255, 0.9);
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.search-input:focus {
  outline: none;
  box-shadow: 0 3px 15px rgba(0, 0, 0, 0.15);
}

.search-button {
  position: absolute;
  right: 5px;
  top: 50%;
  transform: translateY(-50%);
  background: linear-gradient(135deg, #6e8efb, #a777e3);
  border: none;
  color: white;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
}

.search-button:hover {
  transform: translateY(-50%) scale(1.05);
}

.search-icon {
  font-size: 1.2rem;
}

.suggestions-container {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  margin-top: 5px;
  background: white;
  border-radius: 10px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.15);
  overflow: hidden;
  z-index: 10;
}

.suggestions-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.suggestion-item {
  padding: 12px 20px;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: background 0.2s ease;
}

.suggestion-item:hover {
  background-color: rgba(110, 142, 251, 0.1);
}

.suggestion-item:not(:last-child) {
  border-bottom: 1px solid #eee;
}

.suggestion-name {
  font-weight: 500;
}

.suggestion-details {
  color: #666;
  font-size: 0.85rem;
}

.suggestion-loading {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  margin-top: 5px;
  background: white;
  border-radius: 10px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.15);
  padding: 10px;
  text-align: center;
  color: #666;
  font-size: 0.9rem;
}

.geo-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.geo-button {
  background-color: rgba(255, 255, 255, 0.8);
  border: none;
  border-radius: 50px;
  padding: 8px 16px;
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 5px;
}

.geo-button:hover:not(:disabled) {
  background-color: rgba(255, 255, 255, 0.95);
  transform: translateY(-2px);
}

.geo-button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.geo-error {
  color: #e74c3c;
  font-size: 0.85rem;
  text-align: center;
  max-width: 300px;
}
</style> 