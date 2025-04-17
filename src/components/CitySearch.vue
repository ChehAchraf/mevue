<script setup>
import { ref } from 'vue';

const emit = defineEmits(['search', 'geolocation']);
const searchQuery = ref('');
const geoLoading = ref(false);
const geoError = ref(null);

const handleSubmit = (e) => {
  e.preventDefault();
  if (searchQuery.value.trim()) {
    emit('search', searchQuery.value.trim());
  }
};

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
    <form @submit="handleSubmit" class="search-form">
      <div class="search-container">
        <input
          type="text"
          v-model="searchQuery"
          placeholder="Rechercher une ville..."
          class="search-input"
        />
        <button type="submit" class="search-button">
          <span class="search-icon">🔍</span>
        </button>
      </div>
    </form>
    
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

.search-form {
  width: 100%;
  max-width: 500px;
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