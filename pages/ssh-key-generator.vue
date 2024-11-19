<script setup>
import { ref } from 'vue'

const public_key = ref('')
const private_key = ref('')
const downloaded = ref(false)

async function generate() {
    const obj = await fetch('https://xs679698.xsrv.jp/ssh-keygen-laravel/public/generate').then(r => r.json())
    public_key.value = obj.public_key
    private_key.value = obj.private_key
}


onMounted(() => {
    generate()
})

function downloadTextFile(text, filename) {
    const blob = new Blob([text], { type: 'text/plain' })
    const link = document.createElement('a')
    link.download = filename
    link.href = URL.createObjectURL(blob)
    link.click()
}

function download() {
    const prefix = new Date().toLocaleString('sv').replace(' ', 'T').replaceAll(/[^\dT]/g, '')
    downloadTextFile(public_key.value, `${prefix}_public_key.txt`)
    downloadTextFile(private_key.value, `${prefix}_private_key.txt`)
    downloaded.value = true
}

useHead({
    title: 'SSH Key Generator',
})
</script>

<template>
    <v-container>
        <v-btn @click="generate" color="primary">Generate</v-btn>
        <v-btn @click="download" color="secondary" class="ml-4">Download</v-btn>
        <v-textarea label="Public key" v-model="public_key" rows="5" class="mt-4"></v-textarea>
        <v-textarea label="Private key" v-model="private_key" rows="15"></v-textarea>
        <v-snackbar v-model="downloaded">Downloaded</v-snackbar>
    </v-container>
</template>
