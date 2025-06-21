<template>
  <div class="contenedor">
    <h6 class="Titulo">Conversión de Monedas</h6>
    <q-page padding>
      <q-input
        v-model="amount"
        label="Monto a convertir"
        type="number"
        :rules="[(val) => val > 0 || 'Debe ser un monto válido']"
        lazy-rules
      />
      <q-select v-model="fromCurrency" :options="currencies" label="Desde" emit-value map-options />
      <q-select v-model="toCurrency" :options="currencies" label="Hacia" emit-value map-options />

      <div class="boton-container">
        <q-btn label="Convertir" @click="convertCurrency" :disabled="isLoading" color="primary" />
        <q-btn label="Intercambiar Monedas" @click="swapCurrencies" color="purple" />
      </div>
      <q-banner v-if="conversionResult" class="q-mt-md">
        {{ amount }} {{ fromCurrency }} equivalen a {{ conversionResult }} {{ toCurrency }}
      </q-banner>

      <q-banner v-if="errorMessage" class="q-mt-md" color="negative">
        {{ errorMessage }}
      </q-banner>
    </q-page>
  </div>
</template>

<script>
import { ref } from 'vue'
import { api } from 'boot/axios'

export default {
  name: 'CurrencyConverter',
  setup() {
    const amount = ref(0)
    const fromCurrency = ref(null)
    const toCurrency = ref(null)
    const currencies = ref([]) // Lista de monedas
    const conversionResult = ref(null)
    const errorMessage = ref(null)
    const isLoading = ref(false)

    // Función para obtener la lista de monedas
    const fetchCurrencies = async () => {
      try {
        const response = await api.get('/currencies')
        currencies.value = Object.keys(response.data).map((currency) => ({
          label: currency,
          value: currency,
        }))
      } catch {
        errorMessage.value = 'Error al obtener las monedas disponibles'
      }
    }

    // Función para convertir monedas
    const convertCurrency = async () => {
      if (!amount.value || !fromCurrency.value || !toCurrency.value) {
        errorMessage.value = 'Por favor, complete todos los campos.'
        return
      }
      isLoading.value = true
      errorMessage.value = ''
      try {
        const response = await api.get('/latest', {
          params: {
            amount: amount.value,
            from: fromCurrency.value,
            to: toCurrency.value,
          },
        })
        conversionResult.value = response.data.rates[toCurrency.value]
        isLoading.value = false
      } catch {
        errorMessage.value = 'Error al realizar la conversión'
        isLoading.value = false
      }
    }

    // Función para intercambiar las monedas
    const swapCurrencies = () => {
      const temp = fromCurrency.value
      fromCurrency.value = toCurrency.value
      toCurrency.value = temp
    }

    // Llamada inicial para obtener las monedas
    fetchCurrencies()

    return {
      amount,
      fromCurrency,
      toCurrency,
      currencies,
      conversionResult,
      errorMessage,
      isLoading,
      fetchCurrencies,
      convertCurrency,
      swapCurrencies,
    }
  },
}
</script>

<style scoped>
.q-banner {
  font-size: 18px;
  text-align: center;
}

.contenedor {
  width: 50%;
  height: 80vh; /* Ocupar el 80% de la altura de la pantalla (opcional) */
  margin: 30px auto 0;
  padding: 20px;
  border: 2px solid #2c0672; /* Borde de 2px de grosor y color negro */
  border-radius: 10px; /* Bordes redondeados (opcional, puedes ajustarlo) */
  box-sizing: border-box;
}

.boton-container {
  text-align: center; /* Centra los botones */
}

.boton-container button {
  margin: 10px; /* Espacio entre los botones */
}
</style>
