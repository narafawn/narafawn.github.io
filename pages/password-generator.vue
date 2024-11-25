<script setup>
import { ref } from 'vue'

const password = ref('')
const history = ref([])
const useNumber = ref(true)
const useUpper = ref(false)
const useLower = ref(true)
const useSymbol = ref(false)
const useReadable = ref(true)
const length = ref(8)
const copied = ref(false)
const headers = [
    { title: 'Password', key: 'password' },
    { title: 'CreatedAt', key: 'createdAt', value: item => new Date(item.createdAt).toLocaleString('sv') },
]

function getRandomCharacter(str) {
    if (useReadable.value) {
        const unreadableChars = 'I1lO0o'
        str = str.replaceAll(new RegExp(`[${unreadableChars}]`, 'g'), '')
    }

    if (str.length === 0) {
        return ''
    }

    const randomIndex = Math.floor(Math.random() * str.length)
    return str[randomIndex]
}

function generate() {
    const numbers = '0123456789'
    const lowerChars = 'abcdefghijklmnopqrstuvwxyz'
    const upperChars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
    const symbols = decodeURIComponent('!%22%23%24%25%26%27()*%2B%2C%5C-.%2F%3A%3B%3C%3D%3E%3F%40%5B%5C%5D%5E_%60%7B%7C%7D~')
    let generatedPassword = ''
    if (useNumber.value) {
        generatedPassword += getRandomCharacter(numbers)
    }
    if (useUpper.value) {
        generatedPassword += getRandomCharacter(upperChars)
    }
    if (useLower.value) {
        generatedPassword += getRandomCharacter(lowerChars)
    }
    if (useSymbol.value) {
        generatedPassword += getRandomCharacter(symbols)
    }
    const remainLength = length.value - generatedPassword.length
    for (let i = 0; i < remainLength; i++) {
        let chars = ''
        if (useNumber.value) {
            chars += numbers
        }
        if (useUpper.value) {
            chars += upperChars
        }
        if (useLower.value) {
            chars += lowerChars
        }
        if (useSymbol.value) {
            chars += symbols
        }
        generatedPassword += getRandomCharacter(chars)
    }
    // https://stackoverflow.com/a/60963711
    password.value = [...generatedPassword].sort(() => Math.random() - .5).join('')
    history.value = [{ password: password.value, createdAt: new Date() }, ...history.value]
    localStorage.setItem('password.history', JSON.stringify(history.value))
}

function copy() {
    const i = document.body.appendChild(document.createElement('input'))
    i.value = password.value
    i.select()
    document.execCommand('copy')
    document.body.removeChild(i)
    copied.value = true
}

onMounted(() => {
    history.value = JSON.parse(localStorage.getItem('password.history')) ?? []
    generate()
})

function handleClickRow(event, row) {
    const i = document.body.appendChild(document.createElement('input'))
    i.value = row.item.password
    i.select()
    document.execCommand('copy')
    document.body.removeChild(i)
    copied.value = true
}

useHead({
    title: 'Password Generator',
})
</script>

<template>
    <v-container>
        <div class="d-flex">
            <v-checkbox label="123" v-model="useNumber" hide-details></v-checkbox>
            <v-checkbox label="ABC" v-model="useUpper" hide-details></v-checkbox>
            <v-checkbox label="abc" v-model="useLower" hide-details></v-checkbox>
            <v-checkbox label="#$&" v-model="useSymbol" hide-details></v-checkbox>
            <v-checkbox label="Readable" v-model="useReadable" hide-details></v-checkbox>
            <v-text-field type="number" label="Length" v-model.number="length" min="0" class="flex-grow-0 ml-2"
                width="200" density="compact" hide-details />
        </div>
        <v-btn @click="generate" color="primary">Generate</v-btn>
        <div class="d-flex align-center mt-4">
            <v-text-field v-model="password" label="Password" class="flex-grow-0" width="200px"
                hide-details></v-text-field>
            <v-btn @click="copy" color="secondary" class="ml-4">Copy</v-btn>
        </div>
        <v-data-table :headers="headers" :items="history" @click:row="handleClickRow"></v-data-table>
        <v-snackbar v-model="copied">Copied!</v-snackbar>
    </v-container>
</template>