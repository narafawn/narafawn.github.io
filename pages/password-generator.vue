<script setup>
import { ref } from 'vue'

const password = ref('')
const history = ref([])
const copied = ref(false)
const headers = [
    { title: 'Password', key: 'password' },
    { title: 'CreatedAt', key: 'createdAt', value: item => new Date(item.createdAt) },
]

function generate() {
    password.value = Math.random().toString(16).slice(-8)
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
        <v-btn @click="generate" color="primary">Generate</v-btn>
        <v-btn @click="copy" color="secondary" class="ml-4">Copy</v-btn>
        <v-text-field v-model="password" label="Password" class="mt-8" width="200px"></v-text-field>
        <v-data-table :headers="headers" :items="history" @click:row="handleClickRow"></v-data-table>
        <v-snackbar v-model="copied">Copied!</v-snackbar>
    </v-container>
</template>