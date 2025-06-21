<template>
  <q-page padding>
    <q-input
      v-model="amount"
      label="Monto"
      type="number"
      :rules="[(val) => val > 0 || 'Debe ser un monto válido']"
      lazy-rules
    />
    <q-select
      v-model="fromCurrency"
      :options="currencies"
      label="Moneda de origen"
      emit-value
      map-options
    />
    <q-select
      v-model="toCurrency"
      :options="currencies"
      label="Moneda de destino"
      emit-value
      map-options
    />
    <q-btn label="Convertir" @click="convertCurrency" :disabled="isLoading" />

    <q-btn label="Intercambiar Monedas" @click="swapCurrencies" />

    <q-banner v-if="conversionResult" class="q-mt-md">
      {{ amount }} {{ fromCurrency }} equivalen a {{ conversionResult }} {{ toCurrency }}
    </q-banner>

    <q-banner v-if="errorMessage" class="q-mt-md" color="negative">
      {{ errorMessage }}
    </q-banner>
  </q-page>
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
</style>
