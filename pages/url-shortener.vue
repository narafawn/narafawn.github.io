<script setup>
import { ref } from 'vue'

const url = ref('')
const name = ref('')
const copied = ref(false)

function debounce(func, delay) {
    let id
    return function (...args) {
        if (id) {
            clearTimeout(id)
        }

        id = setTimeout(() => {
            func.apply(this, args)
        }, delay)
    };
}

const createUrl = debounce(() => fetch(`https://${name.value}.mytoolkit.app`, { method: 'POST', body: url.value }), 1000)

async function handleInput() {
    try {
        new URL(url.value)
        if (!name.value) name.value = Math.random().toString(16).slice(-5)
        createUrl()
    } catch (error) { }
}

async function handleInputName() {
    try {
        new URL(url.value)
        createUrl()
    } catch (error) { }
}

function copy() {
    navigator.clipboard.writeText(`https://${name.value}.mytoolkit.app`)
    copied.value = true
}

useHead({
    title: 'URL Shortener',
})
</script>

<template>
    <v-container>
        <v-text-field v-model="url" label="URL" @input="handleInput"></v-text-field>
        <v-text-field v-if="url" v-model="name" label="Name" @input="handleInputName"></v-text-field>
        <a v-if="url && name" :href="`https://${name}.mytoolkit.app`">https://{{ name }}.mytoolkit.app</a>
        <v-btn v-if="url && name" @click="copy" color="primary" class="ml-4">Copy</v-btn>
        <v-snackbar v-model="copied">Copied!</v-snackbar>
    </v-container>
</template>
