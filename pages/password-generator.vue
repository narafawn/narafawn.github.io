<script setup>
import { ref } from 'vue'

const password = ref(Math.random().toString(16).slice(-8))
const copied = ref(false)

function generate() {
    password.value = Math.random().toString(16).slice(-8)
}

function copy() {
    const i = document.body.appendChild(document.createElement('input'))
    i.value = password.value
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
        <v-snackbar v-model="copied">Copied!</v-snackbar>
    </v-container>
</template>