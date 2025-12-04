<template>
  <div class="app-container">
    <header class="header">
      <h1>Clima</h1>
    </header>

    <div class="search-container">
      <div class="search-box">
        <svg class="search-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="11" cy="11" r="8"></circle>
          <path d="m21 21-4.35-4.35"></path>
        </svg>
        <input
          v-model="cityName"
          @input="onCityInput"
          @keyup.enter="handleEnterKey"
          @focus="showSuggestions = true"
          @blur="handleBlur"
          type="text"
          placeholder="Buscar cidade"
          class="search-input"
        />
        <div v-if="loadingSuggestions" class="suggestions-loading">
          <div class="mini-spinner"></div>
        </div>
        <div v-if="showSuggestions && citySuggestions.length > 0" class="suggestions-list">
          <div
            v-for="(city, index) in citySuggestions"
            :key="index"
            @mousedown="selectCity(city)"
            class="suggestion-item"
          >
            <span class="suggestion-name">{{ city.name }}</span>
            <span class="suggestion-location">{{ city.state ? city.state + ', ' : '' }}{{ city.country }}</span>
          </div>
        </div>
      </div>
    </div>

    <div v-if="loading" class="loading">
      <div class="spinner"></div>
      <p>Carregando informações do clima...</p>
    </div>

    <div v-if="error" class="error-message">
      <svg class="error-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <circle cx="12" cy="12" r="10"></circle>
        <line x1="12" y1="8" x2="12" y2="12"></line>
        <line x1="12" y1="16" x2="12.01" y2="16"></line>
      </svg>
      <p>{{ error }}</p>
    </div>

    <div v-if="currentWeather && !loading" class="weather-container">
      <!-- Clima Atual -->
      <div class="current-weather">
        <div class="location-info">
          <h2>{{ currentWeather.name }}</h2>
          <p class="date">{{ formatDate(currentWeather.dt) }}</p>
        </div>

        <div class="main-weather">
          <div class="temp-section">
            <span class="temp-value">{{ Math.round(currentWeather.main.temp) }}°</span>
            <div class="temp-range">
              <span class="temp-max">{{ Math.round(currentWeather.main.temp_max) }}°</span>
              <span class="temp-separator">/</span>
              <span class="temp-min">{{ Math.round(currentWeather.main.temp_min) }}°</span>
            </div>
          </div>
          <div class="icon-section">
            <img
              :src="`https://openweathermap.org/img/wn/${currentWeather.weather[0].icon}@2x.png`"
              :alt="currentWeather.weather[0].description"
              class="weather-icon"
            />
            <p class="weather-description">
              {{ capitalizeFirst(currentWeather.weather[0].description) }}
            </p>
          </div>
        </div>

        <div class="weather-details">
          <div class="detail-item">
            <svg class="detail-icon" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M14 14.76V3.5a2.5 2.5 0 0 0-5 0v11.26a4.5 4.5 0 1 0 5 0z"></path>
            </svg>
            <div class="detail-content">
              <span class="detail-label">Sensação</span>
              <span class="detail-value">{{ Math.round(currentWeather.main.feels_like) }}°</span>
            </div>
          </div>
          <div class="detail-item">
            <svg class="detail-icon" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M12 2.69l5.66 5.66a8 8 0 1 1-11.31 0L12 2.69z"></path>
            </svg>
            <div class="detail-content">
              <span class="detail-label">Umidade</span>
              <span class="detail-value">{{ currentWeather.main.humidity }}%</span>
            </div>
          </div>
          <div class="detail-item">
            <svg class="detail-icon" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M9.59 4.59A2 2 0 1 1 11 8H2m10.59 11.41A2 2 0 1 0 14 16H2m15.73-8.27A2.5 2.5 0 1 1 19.5 12H2"></path>
            </svg>
            <div class="detail-content">
              <span class="detail-label">Vento</span>
              <span class="detail-value">{{ Math.round(currentWeather.wind.speed * 3.6) }} km/h</span>
            </div>
          </div>
          <div class="detail-item">
            <svg class="detail-icon" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <circle cx="12" cy="12" r="10"></circle>
              <path d="M12 6v6l4 2"></path>
            </svg>
            <div class="detail-content">
              <span class="detail-label">Pressão</span>
              <span class="detail-value">{{ currentWeather.main.pressure }} hPa</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Previsão dos Próximos Dias -->
      <div v-if="forecast.length > 0" class="forecast-container">
        <div class="forecast-grid">
          <div
            v-for="(day, index) in forecast"
            :key="index"
            class="forecast-card"
          >
            <p class="forecast-date">{{ formatForecastDate(day.dt) }}</p>
            <img
              :src="`https://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`"
              :alt="day.weather[0].description"
              class="forecast-icon"
            />
            <div class="forecast-temps">
              <span class="forecast-temp-max">{{ Math.round(day.main.temp_max) }}°</span>
              <span class="forecast-temp-min">{{ Math.round(day.main.temp_min) }}°</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="!currentWeather && !loading && !error" class="welcome-message">
      <svg class="welcome-icon" width="64" height="64" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
        <path d="M18 10h-1.26A8 8 0 1 0 9 20h9a5 5 0 0 0 0-10z"></path>
      </svg>
      <p>Digite o nome de uma cidade para ver o clima</p>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',
  data() {
    return {
      cityName: '',
      currentWeather: null,
      forecast: [],
      loading: false,
      error: null,
      apiKey: '4e13fa636e0c6863d6a6f97c414bf39d',
      citySuggestions: [],
      loadingSuggestions: false,
      showSuggestions: false,
      searchTimeout: null
    }
  },
  methods: {
    onCityInput() {
      // Limpar timeout anterior
      if (this.searchTimeout) {
        clearTimeout(this.searchTimeout)
      }

      // Se o campo estiver vazio, limpar sugestões
      if (!this.cityName.trim()) {
        this.citySuggestions = []
        this.showSuggestions = false
        return
      }

      // Debounce: aguardar 300ms após parar de digitar
      this.searchTimeout = setTimeout(() => {
        this.searchCitySuggestions()
      }, 300)
    },
    async searchCitySuggestions() {
      if (!this.cityName.trim() || this.cityName.trim().length < 2) {
        this.citySuggestions = []
        return
      }

      this.loadingSuggestions = true
      this.showSuggestions = true

      try {
        const encodedCityName = encodeURIComponent(this.cityName.trim())
        const response = await axios.get(
          `https://api.openweathermap.org/geo/1.0/direct?q=${encodedCityName}&limit=5&appid=${this.apiKey}`
        )
        this.citySuggestions = response.data
      } catch (err) {
        // Silenciosamente falhar nas sugestões para não interromper a experiência
        this.citySuggestions = []
        console.error('Erro ao buscar sugestões:', err)
      } finally {
        this.loadingSuggestions = false
      }
    },
    selectCity(city) {
      // Usar apenas o nome da cidade para a busca
      const cityToSearch = city.name
      this.cityName = `${city.name}${city.state ? ', ' + city.state : ''}, ${city.country}`
      this.citySuggestions = []
      this.showSuggestions = false
      // Buscar clima usando o nome da cidade
      this.searchWeatherWithCity(cityToSearch)
    },
    async searchWeatherWithCity(cityName) {
      if (!cityName || !cityName.trim()) {
        this.error = 'Por favor, digite o nome de uma cidade.'
        return
      }

      if (!this.apiKey || this.apiKey === 'YOUR_API_KEY') {
        this.error = 'Chave da API não configurada. Por favor, configure sua chave da API no código.'
        return
      }

      this.loading = true
      this.error = null
      this.currentWeather = null
      this.forecast = []

      try {
        const encodedCityName = encodeURIComponent(cityName.trim())
        
        const currentResponse = await axios.get(
          `https://api.openweathermap.org/data/2.5/weather?q=${encodedCityName}&appid=${this.apiKey}&units=metric&lang=pt_br`
        )
        this.currentWeather = currentResponse.data

        const forecastResponse = await axios.get(
          `https://api.openweathermap.org/data/2.5/forecast?q=${encodedCityName}&appid=${this.apiKey}&units=metric&lang=pt_br`
        )
        
        const dailyForecast = this.groupForecastByDay(forecastResponse.data.list)
        this.forecast = dailyForecast.slice(1, 6)
      } catch (err) {
        if (err.response) {
          const errorData = err.response.data
          const errorMessage = errorData?.message || ''
          
          if (err.response.status === 404) {
            this.error = 'Cidade não encontrada. Verifique o nome e tente novamente.'
          } else if (err.response.status === 401) {
            if (errorMessage.includes('Invalid API key')) {
              this.error = 'Chave da API inválida. Verifique se copiou a chave corretamente.'
            } else if (errorMessage.includes('Please note that using API')) {
              this.error = 'Chave da API ainda não foi ativada. Aguarde algumas horas após criar a conta.'
            } else {
              this.error = `Erro de autenticação: ${errorMessage || 'Chave da API inválida ou não ativada. Pode levar até 2 horas para ativar após criar a conta.'}`
            }
            console.error('Detalhes do erro 401:', errorMessage)
            console.error('Resposta completa:', errorData)
          } else if (err.response.status === 429) {
            this.error = 'Muitas requisições. Aguarde um momento e tente novamente.'
          } else {
            this.error = `Erro ${err.response.status}: ${errorMessage || 'Erro ao buscar informações do clima.'}`
          }
        } else if (err.request) {
          this.error = 'Sem resposta do servidor. Verifique sua conexão com a internet.'
        } else {
          this.error = 'Erro ao processar a requisição. Tente novamente.'
        }
        console.error('Erro detalhado:', err)
      } finally {
        this.loading = false
      }
    },
    handleEnterKey() {
      if (this.citySuggestions.length > 0) {
        // Se houver sugestões, selecionar a primeira
        this.selectCity(this.citySuggestions[0])
      } else {
        // Caso contrário, buscar normalmente
        this.searchWeather()
      }
    },
    handleBlur() {
      // Aguardar um pouco antes de esconder para permitir clique nas sugestões
      setTimeout(() => {
        this.showSuggestions = false
      }, 200)
    },
    async searchWeather() {
      // Extrair apenas o nome da cidade (antes da primeira vírgula, se houver)
      const cityToSearch = this.cityName.split(',')[0].trim()
      
      if (!cityToSearch) {
        this.error = 'Por favor, digite o nome de uma cidade.'
        return
      }

      if (!this.apiKey || this.apiKey === 'YOUR_API_KEY') {
        this.error = 'Chave da API não configurada. Por favor, configure sua chave da API no código.'
        return
      }

      this.loading = true
      this.error = null
      this.currentWeather = null
      this.forecast = []

      try {
        const encodedCityName = encodeURIComponent(cityToSearch)
        
        const currentResponse = await axios.get(
          `https://api.openweathermap.org/data/2.5/weather?q=${encodedCityName}&appid=${this.apiKey}&units=metric&lang=pt_br`
        )
        this.currentWeather = currentResponse.data

        const forecastResponse = await axios.get(
          `https://api.openweathermap.org/data/2.5/forecast?q=${encodedCityName}&appid=${this.apiKey}&units=metric&lang=pt_br`
        )
        
        const dailyForecast = this.groupForecastByDay(forecastResponse.data.list)
        this.forecast = dailyForecast.slice(1, 6)
      } catch (err) {
        if (err.response) {
          const errorData = err.response.data
          const errorMessage = errorData?.message || ''
          
          if (err.response.status === 404) {
            this.error = 'Cidade não encontrada. Verifique o nome e tente novamente.'
          } else if (err.response.status === 401) {
            if (errorMessage.includes('Invalid API key')) {
              this.error = 'Chave da API inválida. Verifique se copiou a chave corretamente.'
            } else if (errorMessage.includes('Please note that using API')) {
              this.error = 'Chave da API ainda não foi ativada. Aguarde algumas horas após criar a conta.'
            } else {
              this.error = `Erro de autenticação: ${errorMessage || 'Chave da API inválida ou não ativada. Pode levar até 2 horas para ativar após criar a conta.'}`
            }
            console.error('Detalhes do erro 401:', errorMessage)
            console.error('Resposta completa:', errorData)
          } else if (err.response.status === 429) {
            this.error = 'Muitas requisições. Aguarde um momento e tente novamente.'
          } else {
            this.error = `Erro ${err.response.status}: ${errorMessage || 'Erro ao buscar informações do clima.'}`
          }
        } else if (err.request) {
          this.error = 'Sem resposta do servidor. Verifique sua conexão com a internet.'
        } else {
          this.error = 'Erro ao processar a requisição. Tente novamente.'
        }
        console.error('Erro detalhado:', err)
      } finally {
        this.loading = false
      }
    },
    groupForecastByDay(forecastList) {
      const grouped = {}
      forecastList.forEach(item => {
        const date = new Date(item.dt * 1000)
        const dateKey = date.toDateString()
        if (!grouped[dateKey]) {
          grouped[dateKey] = item
        }
      })
      return Object.values(grouped)
    },
    formatDate(timestamp) {
      const date = new Date(timestamp * 1000)
      const options = {
        weekday: 'long',
        day: 'numeric',
        month: 'long'
      }
      return date.toLocaleDateString('pt-BR', options)
    },
    formatForecastDate(timestamp) {
      const date = new Date(timestamp * 1000)
      const today = new Date()
      const tomorrow = new Date(today)
      tomorrow.setDate(tomorrow.getDate() + 1)

      if (date.toDateString() === tomorrow.toDateString()) {
        return 'Amanhã'
      }

      const options = { weekday: 'short', day: 'numeric' }
      return date.toLocaleDateString('pt-BR', options)
    },
    capitalizeFirst(str) {
      return str.charAt(0).toUpperCase() + str.slice(1)
    }
  }
}
</script>

<style scoped>
.app-container {
  max-width: 1200px;
  margin: 0 auto;
}

.header {
  text-align: center;
  margin-bottom: 30px;
  padding: 20px 0;
}

.header h1 {
  font-size: 2.5rem;
  font-weight: 300;
  color: #2c3e50;
  letter-spacing: 2px;
}

.search-container {
  margin-bottom: 30px;
}

.search-box {
  position: relative;
  max-width: 500px;
  margin: 0 auto;
}

.search-icon {
  position: absolute;
  left: 20px;
  top: 50%;
  transform: translateY(-50%);
  color: #7f8c8d;
  pointer-events: none;
  z-index: 2;
}

.search-input {
  width: 100%;
  padding: 16px 20px 16px 50px;
  border: none;
  border-radius: 30px;
  font-size: 1rem;
  outline: none;
  background: white;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  color: #2c3e50;
}

.search-input:focus {
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

.search-input::placeholder {
  color: #95a5a6;
}

.suggestions-loading {
  position: absolute;
  right: 20px;
  top: 50%;
  transform: translateY(-50%);
  z-index: 2;
}

.mini-spinner {
  border: 2px solid rgba(44, 62, 80, 0.1);
  border-top: 2px solid #3498db;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  animation: spin 1s linear infinite;
}

.suggestions-list {
  position: absolute;
  top: calc(100% + 8px);
  left: 0;
  right: 0;
  background: white;
  border-radius: 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  max-height: 300px;
  overflow-y: auto;
  z-index: 1000;
  margin-top: 4px;
}

.suggestion-item {
  padding: 14px 20px;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  gap: 4px;
  transition: background-color 0.2s ease;
  border-bottom: 1px solid #ecf0f1;
}

.suggestion-item:first-child {
  border-top-left-radius: 16px;
  border-top-right-radius: 16px;
}

.suggestion-item:last-child {
  border-bottom: none;
  border-bottom-left-radius: 16px;
  border-bottom-right-radius: 16px;
}

.suggestion-item:hover {
  background-color: #f8f9fa;
}

.suggestion-name {
  font-size: 1rem;
  font-weight: 500;
  color: #2c3e50;
}

.suggestion-location {
  font-size: 0.85rem;
  color: #7f8c8d;
}

/* Scrollbar personalizada para a lista de sugestões */
.suggestions-list::-webkit-scrollbar {
  width: 6px;
}

.suggestions-list::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 16px;
}

.suggestions-list::-webkit-scrollbar-thumb {
  background: #bdc3c7;
  border-radius: 16px;
}

.suggestions-list::-webkit-scrollbar-thumb:hover {
  background: #95a5a6;
}

.loading {
  text-align: center;
  padding: 60px 20px;
}

.spinner {
  border: 3px solid rgba(44, 62, 80, 0.1);
  border-top: 3px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
  margin: 0 auto 20px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loading p {
  color: #7f8c8d;
  font-size: 0.95rem;
}

.error-message {
  background: #fff;
  border-left: 4px solid #e74c3c;
  border-radius: 8px;
  padding: 16px 20px;
  display: flex;
  align-items: center;
  gap: 12px;
  color: #e74c3c;
  margin: 20px;
  font-size: 0.95rem;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.error-icon {
  flex-shrink: 0;
  color: #e74c3c;
}

.welcome-message {
  text-align: center;
  padding: 80px 20px;
  color: #7f8c8d;
}

.welcome-icon {
  width: 64px;
  height: 64px;
  margin: 0 auto 20px;
  opacity: 0.4;
  color: #7f8c8d;
  display: block;
}

.welcome-message p {
  font-size: 1.1rem;
  font-weight: 300;
}

.weather-container {
  animation: fadeIn 0.5s ease-in;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.current-weather {
  background: white;
  border-radius: 24px;
  padding: 40px;
  margin-bottom: 24px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
}

.location-info {
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 1px solid #ecf0f1;
}

.location-info h2 {
  font-size: 1.8rem;
  font-weight: 400;
  color: #2c3e50;
  margin-bottom: 8px;
}

.date {
  color: #7f8c8d;
  font-size: 0.95rem;
  font-weight: 300;
}

.main-weather {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 40px;
  padding-bottom: 30px;
  border-bottom: 1px solid #ecf0f1;
}

.temp-section {
  display: flex;
  flex-direction: column;
}

.temp-value {
  font-size: 5.5rem;
  font-weight: 200;
  color: #2c3e50;
  line-height: 1;
  margin-bottom: 10px;
}

.temp-range {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 1.1rem;
  color: #7f8c8d;
}

.temp-max {
  font-weight: 500;
  color: #2c3e50;
}

.temp-separator {
  opacity: 0.5;
}

.temp-min {
  opacity: 0.7;
}

.icon-section {
  text-align: center;
}

.weather-icon {
  width: 100px;
  height: 100px;
  margin-bottom: 10px;
}

.weather-description {
  font-size: 1rem;
  color: #7f8c8d;
  text-transform: capitalize;
  font-weight: 300;
}

.weather-details {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 16px;
}

.detail-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px;
  background: #f8f9fa;
  border-radius: 12px;
}

.detail-icon {
  width: 24px;
  height: 24px;
  flex-shrink: 0;
  color: #3498db;
}

.detail-content {
  display: flex;
  flex-direction: column;
}

.detail-label {
  font-size: 0.75rem;
  color: #95a5a6;
  margin-bottom: 4px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.detail-value {
  font-size: 1.1rem;
  font-weight: 500;
  color: #2c3e50;
}

.forecast-container {
  background: white;
  border-radius: 24px;
  padding: 32px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
}

.forecast-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 16px;
}

.forecast-card {
  background: #f8f9fa;
  border-radius: 16px;
  padding: 20px;
  text-align: center;
  transition: all 0.3s ease;
}

.forecast-card:hover {
  background: #ecf0f1;
  transform: translateY(-2px);
}

.forecast-date {
  font-size: 0.85rem;
  color: #7f8c8d;
  margin-bottom: 12px;
  font-weight: 400;
  text-transform: capitalize;
}

.forecast-icon {
  width: 50px;
  height: 50px;
  margin: 12px 0;
}

.forecast-temps {
  display: flex;
  justify-content: center;
  align-items: baseline;
  gap: 8px;
  margin-top: 8px;
}

.forecast-temp-max {
  font-size: 1.3rem;
  font-weight: 500;
  color: #2c3e50;
}

.forecast-temp-min {
  font-size: 1rem;
  color: #95a5a6;
  font-weight: 300;
}

@media (max-width: 768px) {
  .header h1 {
    font-size: 2rem;
  }

  .current-weather {
    padding: 24px;
  }

  .main-weather {
    flex-direction: column;
    gap: 30px;
    text-align: center;
  }

  .temp-value {
    font-size: 4.5rem;
  }

  .weather-details {
    grid-template-columns: repeat(2, 1fr);
  }

  .forecast-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 480px) {
  body {
    padding: 10px;
  }

  .header h1 {
    font-size: 1.8rem;
  }

  .current-weather {
    padding: 20px;
  }

  .temp-value {
    font-size: 4rem;
  }

  .weather-details {
    grid-template-columns: 1fr;
  }

  .forecast-grid {
    grid-template-columns: 1fr;
  }
}
</style>

