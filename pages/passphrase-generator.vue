<script setup>
import { ref } from 'vue'

const passphrase = ref('')
const history = ref([])
const wordCount = ref(3)
let words
const copied = ref(false)
const headers = [
    { title: 'Passphrase', key: 'passphrase' },
    { title: 'CreatedAt', key: 'createdAt', value: item => new Date(item.createdAt).toLocaleString('sv') },
]

function generate() {
    const wordList = words.split('\n')
    passphrase.value = Array(wordCount.value).fill().map(() => {
        const randomIndex = Math.floor(Math.random() * wordList.length)
        return wordList[randomIndex]
    }).join('-')
    history.value = [{ passphrase: passphrase.value, createdAt: new Date() }, ...history.value]
    localStorage.setItem('passphrase.history', JSON.stringify(history.value))
}

function handleWordCountChange() {
    localStorage.setItem('passphrase-word-count', wordCount.value)
    generate()
}

function copy() {
    const i = document.body.appendChild(document.createElement('input'))
    i.value = passphrase.value
    i.select()
    document.execCommand('copy')
    document.body.removeChild(i)
    copied.value = true
}

onMounted(async () => {
    const wordCountValue = localStorage.getItem('passphrase-word-count')
    if (wordCountValue) {
        wordCount.value = parseInt(wordCountValue, 10)
    }
    words = localStorage.getItem('passphrase-words')
    if (!words) {
        words = await fetch('https://ws.vercel.app/eff_large_wordlist.txt').then(r => r.text())
        localStorage.setItem('passphrase-words', words)
    }
    history.value = JSON.parse(localStorage.getItem('passphrase.history')) ?? []
    generate()
})

function handleClickRow(event, row) {
    const i = document.body.appendChild(document.createElement('input'))
    i.value = row.item.passphrase
    i.select()
    document.execCommand('copy')
    document.body.removeChild(i)
    copied.value = true
}

useHead({
    title: 'Passphrase Generator',
})
</script>

<template>
    <v-container>
        <v-text-field type="number" label="Word Count" v-model.number="wordCount" min="0" width="200"
            @change="handleWordCountChange" />
        <v-btn @click="generate" color="primary">Generate</v-btn>
        <v-btn @click="copy" color="secondary" class="ml-4">Copy</v-btn>
        <v-text-field v-model="passphrase" label="Passphrase" class="mt-8" width="350"></v-text-field>
        <v-data-table :headers="headers" :items="history" @click:row="handleClickRow"></v-data-table>
        <v-snackbar v-model="copied">Copied!</v-snackbar>
    </v-container>
</template>