<script setup>
import { ref } from 'vue'

function generateUUID() {
    return ([1e7] + -1e3 + -4e3 + -8e3 + -1e11).replace(/[018]/g, c =>
        (c ^ crypto.getRandomValues(new Uint8Array(1))[0] & 15 >> c / 4).toString(16)
    );
}

const uuid = ref(generateUUID())
const copied = ref(false)

function generate() {
    uuid.value = generateUUID()
}

function copy() {
    const i = document.body.appendChild(document.createElement('input'))
    i.value = uuid.value
    i.select()
    document.execCommand('copy')
    document.body.removeChild(i)
    copied.value = true
}

useHead({
    title: 'UUID Generator',
})
</script>

<template>
    <v-container>
        <v-btn @click="generate" color="primary">Generate</v-btn>
        <v-btn @click="copy" color="secondary" class="ml-4">Copy</v-btn>
        <v-text-field v-model="uuid" label="UUID" class="mt-8" style="max-width: 500px;"></v-text-field>
        <v-snackbar v-model="copied">Copied!</v-snackbar>
    </v-container>
</template>