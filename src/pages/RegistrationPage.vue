<template>
  <div class="form-page">
    <h1>Регистрация</h1>
    <form @submit.prevent="handleSubmit" class="registration-form">
      <!-- Имя -->
      <div class="form-group">
        <label for="name">Имя</label>
        <input
          id="name"
          v-model="form.name"
          type="text"
          class="form-input"
          :class="{ 'input-error': errors.name }"
          @blur="validateField('name')"
        />
        <span v-if="errors.name" class="error-message">{{ errors.name }}</span>
      </div>

      <!-- Email -->
      <div class="form-group">
        <label for="email">Email</label>
        <input
          id="email"
          v-model="form.email"
          type="email"
          class="form-input"
          :class="{ 'input-error': errors.email }"
          @blur="validateField('email')"
        />
        <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
      </div>

      <!-- Пароль -->
      <div class="form-group">
        <label for="password">Пароль</label>
        <input
          id="password"
          v-model="form.password"
          type="password"
          class="form-input"
          :class="{ 'input-error': errors.password }"
          @blur="validateField('password')"
        />
        <span v-if="errors.password" class="error-message">{{ errors.password }}</span>
      </div>

      <!-- Подтверждение пароля -->
      <div class="form-group">
        <label for="confirmPassword">Подтверждение пароля</label>
        <input
          id="confirmPassword"
          v-model="form.confirmPassword"
          type="password"
          class="form-input"
          :class="{ 'input-error': errors.confirmPassword }"
          @blur="validateField('confirmPassword')"
        />
        <span v-if="errors.confirmPassword" class="error-message">{{ errors.confirmPassword }}</span>
      </div>

      <div v-if="generalError" class="general-error">{{ generalError }}</div>

      <button type="submit" class="btn-submit" :disabled="isLoading">
        {{ isLoading ? 'Отправка...' : 'Зарегистрироваться' }}
      </button>
    </form>

    <div v-if="successMessage" class="success-message">
      {{ successMessage }}
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import axios from 'axios'

const isLoading = ref(false)
const successMessage = ref('')
const generalError = ref(null)

const form = reactive({
  name: '',
  email: '',
  password: '',
  confirmPassword: ''
})

const errors = reactive({
  name: null,
  email: null,
  password: null,
  confirmPassword: null
})

// Живая валидация при потере фокуса
const validateField = (field) => {
  errors[field] = null
  if (field === 'name') {
    if (!form.name || form.name.trim().length < 2) {
      errors.name = 'Имя должно быть не короче 2 символов.'
    }
  }

  if (field === 'email') {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    if (!form.email || !emailRegex.test(form.email)) {
      errors.email = 'Введите корректный email.'
    }
  }

  if (field === 'password') {
    const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8}$/
    if (!form.password || !passwordRegex.test(form.password)) {
      errors.password = 'Пароль от 8 символов, должен содержать заглавные, строчные буквы, цифры и спецсимволы.'
    }
  }

  if (field === 'confirmPassword') {
    if (form.password !== form.confirmPassword) {
      errors.confirmPassword = 'Пароли не совпадают.'
    } else if (!form.confirmPassword && form.password) {
      errors.confirmPassword = 'Подтвердите пароль.'
    }
  }
}

// Полная валидация перед отправкой
const validate = () => {
  Object.keys(errors).forEach(key => validateField(key))
  return !Object.values(errors).some(error => error !== null)
}

// Отправка формы
const handleSubmit = async () => {
  successMessage.value = ''
  generalError.value = null

  if (!validate()) {
    return
  }

  isLoading.value = true

  try {
    const payload = {
      name: form.name.trim(),
      email: form.email.trim(),
      password: form.password,
      confirmPassword: form.confirmPassword
    }

    // Отправка через Axios на тестовый сервер
    const response = await axios.post('https://jsonplaceholder.typicode.com/users', payload)
    successMessage.value = 'Регистрация успешно отправлена (демо). Данные: ' + JSON.stringify(response.data, null, 2)
    
  } catch (err) {
    // Обработка ошибок Axios (Пункт 9)
    if (err.response) {
      generalError.value = `Ошибка сервера: ${err.response.status}`
    } else if (err.request) {
      generalError.value = 'Ошибка сети: не удалось связаться с сервером.'
    } else {
      generalError.value = 'Произошла ошибка при отправке.'
    }
  } finally {
    isLoading.value = false
  }
}
</script>