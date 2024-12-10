<template>
  <div class="min-h-screen bg-gray-100 flex items-center justify-center p-4">
    <div class="bg-white shadow-md rounded-lg p-6 w-full max-w-md">
      <h1 class="text-2xl font-bold mb-4 text-center">Password Validator</h1>
      <form @submit.prevent="validatePassword">
        <div class="mb-4">
          <label for="password" class="block text-sm font-medium text-gray-700">Password</label>
          <div class="flex">
            <input
              v-model="password"
              :type="showPassword ? 'text' : 'password'"
              id="password"
              class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
              placeholder="Enter your password"
              required
            />
            <button
              type="button"
              @click="togglePasswordVisibility"
              class="ml-2 bg-gray-200 text-gray-700 px-4 py-2 rounded-md hover:bg-gray-300 focus:ring-2 focus:ring-indigo-500 focus:ring-opacity-50"
            >
              {{ showPassword ? 'Hide' : 'Show' }}
            </button>
          </div>
        </div>
        <div class="mb-4">
          <label for="length" class="block text-sm font-medium text-gray-700"
            >Password Length: {{ passwordLength }}</label
          >
          <input type="range" v-model="passwordLength" min="8" max="20" class="mt-1 w-full" />
        </div>
        <button
          type="button"
          @click="generatePassword"
          class="w-full bg-indigo-600 text-white py-2 rounded-md hover:bg-indigo-700 focus:ring-2 focus:ring-indigo-500 focus:ring-opacity-50"
        >
          Generate Random Password
        </button>
        <button
          type="submit"
          class="mt-2 w-full bg-indigo-600 text-white py-2 rounded-md hover:bg-indigo-700 focus:ring-2 focus:ring-indigo-500 focus:ring-opacity-50"
        >
          Validate
        </button>
        <button
          type="button"
          @click="saveToHistory"
          class="mt-2 w-full bg-green-600 text-white py-2 rounded-md hover:bg-green-700 focus:ring-2 focus:ring-green-500 focus:ring-opacity-50"
        >
          Save to History
        </button>
        <button
          type="button"
          @click="showHistory"
          class="mt-2 w-full bg-blue-600 text-white py-2 rounded-md hover:bg-blue-700 focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50"
        >
          Show History
        </button>
        <button
          type="button"
          @click="clearPassword"
          class="mt-2 w-full bg-red-600 text-white py-2 rounded-md hover:bg-red-700 focus:ring-2 focus:ring-red-500 focus:ring-opacity-50"
        >
          Clear Password
        </button>
        <button
          type="button"
          @click="clearHistory"
          class="mt-2 w-full bg-red-800 text-white py-2 rounded-md hover:bg-red-900 focus:ring-2 focus:ring-red-500 focus:ring-opacity-50"
        >
          Clear History
        </button>
      </form>
      <div v-if="password" class="mt-6">
        <h2 class="text-xl font-bold text-center">Validation Results:</h2>
        <ul class="list-disc pl-5 mt-2">
          <li
            v-for="(result, index) in validationResults"
            :key="index"
            :class="result.valid ? 'text-green-500' : 'text-red-500'"
          >
            {{ result.message }}
          </li>
        </ul>
        <div class="mt-4">
          <h3 class="text-lg font-semibold">
            Password Strength:
            <span :class="passwordStrength.color">{{ passwordStrength.level }}</span>
          </h3>
          <div class="w-full h-2 mt-1 rounded bg-gray-200">
            <div
              :style="{
                width:
                  passwordStrength.level === 'Weak'
                    ? '33%'
                    : passwordStrength.level === 'Moderate'
                      ? '66%'
                      : '100%'
              }"
              class="h-full rounded"
              :class="
                passwordStrength.level === 'Weak'
                  ? 'bg-red-500'
                  : passwordStrength.level === 'Moderate'
                    ? 'bg-yellow-500'
                    : 'bg-green-500'
              "
            ></div>
          </div>
        </div>
        <div v-if="passwordTips.length" class="mt-4">
          <h3 class="text-lg font-semibold">Tips to Improve:</h3>
          <ul class="list-disc pl-5">
            <li v-for="(tip, index) in passwordTips" :key="index">{{ tip }}</li>
          </ul>
        </div>
        <div v-if="breachResult" class="mt-4">
          <h3 class="text-lg font-semibold text-red-500">
            Warning: Your password has appeared in a data breach!
          </h3>
        </div>
        <div v-if="commonPasswordWarning" class="mt-4">
          <h3 class="text-lg font-semibold text-red-500">
            Warning: Your password is too common or easy to guess!
          </h3>
          <p class="text-sm text-gray-600">
            Consider using a mix of letters, numbers, and symbols to increase security.
          </p>
        </div>
        <button
          v-if="password"
          @click="copyToClipboard"
          class="mt-4 w-full bg-gray-300 text-gray-800 py-2 rounded-md hover:bg-gray-400 focus:ring-2 focus:ring-indigo-500 focus:ring-opacity-50"
        >
          Copy to Clipboard
        </button>
      </div>
      <div v-if="history.length" class="mt-6">
        <h2 class="text-xl font-bold text-center">Password History:</h2>
        <ul class="list-disc pl-5 mt-2">
          <li v-for="(item, index) in history" :key="index">{{ item }}</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      password: '',
      validationResults: [],
      passwordStrength: {
        level: '',
        color: ''
      },
      passwordTips: [],
      breachResult: false,
      commonPasswordWarning: false,
      showPassword: false, // Flag to toggle password visibility
      passwordLength: 12, // Default password length
      history: [] // Array to store password history
    }
  },
  mounted() {
    this.loadHistory() // Load password history on component mount
  },
  methods: {
    async validatePassword() {
      // Reset results
      this.validationResults = []
      this.passwordTips = []
      this.breachResult = false
      this.commonPasswordWarning = false
      this.passwordStrength = {
        level: '',
        color: ''
      }

      // Basic validation checks
      const lengthValid = this.password.length >= 8
      const uppercaseValid = /[A-Z]/.test(this.password)
      const lowercaseValid = /[a-z]/.test(this.password)
      const numberValid = /[0-9]/.test(this.password)
      const specialCharValid = /[^A-Za-z0-9]/.test(this.password)

      this.validationResults = [
        { message: 'At least 8 characters long', valid: lengthValid },
        { message: 'Contains an uppercase letter', valid: uppercaseValid },
        { message: 'Contains a lowercase letter', valid: lowercaseValid },
        { message: 'Contains a number', valid: numberValid },
        { message: 'Contains a special character', valid: specialCharValid }
      ]

      // Password strength
      const strengthScore = [
        lengthValid,
        uppercaseValid,
        lowercaseValid,
        numberValid,
        specialCharValid
      ].filter(Boolean).length
      if (strengthScore <= 2) {
        this.passwordStrength = { level: 'Weak', color: 'text-red-500' }
      } else if (strengthScore === 3) {
        this.passwordStrength = { level: 'Moderate', color: 'text-yellow-500' }
      } else {
        this.passwordStrength = { level: 'Strong', color: 'text-green-500' }
      }

      // Tips to improve password
      if (!lengthValid) this.passwordTips.push('Increase the length to at least 8 characters.')
      if (!uppercaseValid) this.passwordTips.push('Add at least one uppercase letter.')
      if (!lowercaseValid) this.passwordTips.push('Add at least one lowercase letter.')
      if (!numberValid) this.passwordTips.push('Add at least one number.')
      if (!specialCharValid) this.passwordTips.push('Add at least one special character.')

      // Check for common or weak passwords
      this.checkCommonPassword(this.password)

      // Check for sequential characters
      this.checkSequentialCharacters(this.password)

      // Check password breach
      await this.checkPasswordBreach()
    },
    checkCommonPassword(password) {
      const commonPasswords = [
        '123456',
        'password',
        '123456789',
        '12345678',
        '12345',
        '1234567',
        'qwerty',
        'abc123',
        'letmein',
        'welcome',
        'admin',
        'passw0rd',
        '123123',
        'football',
        'iloveyou'
      ]

      if (commonPasswords.includes(password.toLowerCase())) {
        this.commonPasswordWarning = true
      }
    },
    checkSequentialCharacters(password) {
      const sequentialPatterns = [
        '123456',
        'abcdef',
        'ghijkl',
        'mnopqr',
        'stuvwx',
        'yz',
        '098765',
        'zyxwv',
        'qwert',
        'asdfg'
      ]

      for (const pattern of sequentialPatterns) {
        if (password.includes(pattern)) {
          this.validationResults.push({
            message: 'Password should not contain sequential characters.',
            valid: false
          })
          break
        }
      }
    },
    async checkPasswordBreach() {
      const hash = await this.sha1(this.password)
      const prefix = hash.slice(0, 5)
      const suffix = hash.slice(5).toUpperCase()

      try {
        const response = await fetch(`https://api.pwnedpasswords.com/range/${prefix}`)
        const data = await response.text()

        if (data.includes(suffix)) {
          this.breachResult = true
        }
      } catch (error) {
        console.error('Error checking password breach:', error)
      }
    },
    async sha1(message) {
      const msgBuffer = new TextEncoder().encode(message)
      const hashBuffer = await crypto.subtle.digest('SHA-1', msgBuffer)
      const hashArray = Array.from(new Uint8Array(hashBuffer))
      const hashHex = hashArray.map((b) => b.toString(16).padStart(2, '0')).join('')
      return hashHex.toUpperCase()
    },
    generatePassword() {
      const length = this.passwordLength // Use the length from the slider
      const charset =
        'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$%^&*()_+~`|}{[]:;?><,./-='
      let password = ''
      for (let i = 0, n = charset.length; i < length; ++i) {
        password += charset.charAt(Math.floor(Math.random() * n))
      }
      this.password = password
    },
    togglePasswordVisibility() {
      this.showPassword = !this.showPassword
    },
    copyToClipboard() {
      navigator.clipboard
        .writeText(this.password)
        .then(() => {
          alert('Password copied to clipboard!')
        })
        .catch((err) => {
          console.error('Failed to copy: ', err)
        })
    },
    saveToHistory() {
      if (this.password) {
        this.history.push(this.password)
        localStorage.setItem('passwordHistory', JSON.stringify(this.history))
      } else {
        alert('No password to save!')
      }
    },
    loadHistory() {
      const storedHistory = localStorage.getItem('passwordHistory')
      if (storedHistory) {
        this.history = JSON.parse(storedHistory)
      }
    },
    showHistory() {
      if (this.history.length === 0) {
        alert('No password history available.')
      } else {
        alert('Password History:\n' + this.history.join('\n'))
      }
    },
    clearPassword() {
      this.password = '' // Clear the password input
      this.validationResults = [] // Reset validation results
      this.passwordStrength = { level: '', color: '' } // Reset password strength
      this.passwordTips = [] // Reset tips
      this.breachResult = false // Reset breach result
      this.commonPasswordWarning = false // Reset common password warning
    },
    clearHistory() {
      this.history = []
      localStorage.removeItem('passwordHistory') // Clear history from localStorage
      alert('Password history cleared!')
    }
  }
}
</script>

<style scoped>
body {
  @apply bg-gray-100;
}
</style>
