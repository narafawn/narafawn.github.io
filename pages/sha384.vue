<script setup>
import { ref } from 'vue'

const text = ref('')
const hash = ref('38b060a751ac96384cd9327eb1b1e36a21fdb71114be07434c0cc7bf63f6e1da274edebfe76f65fbd51ad2f14898b95b')

function doHash(string) {
    hash.value = forge.md.sha384.create().update(string).digest().toHex()
}

async function handleInput() {
    hash.value = ''
    doHash(text.value)
}

function fileToString(file) {
    return new Promise(resolve => {
        const reader = new FileReader()
        reader.onload = () => resolve([...new Uint8Array(reader.result)].map(byte => String.fromCharCode(byte)).join(''))
        reader.readAsArrayBuffer(file)
    })
}

async function handleChange(e) {
    if (!e.target.files.length) return
    hash.value = ''
    const file = e.target.files[0]
    const string = await fileToString(file)
    doHash(string)
}

useHead({
    title: 'SHA384 Hash',
    script: [
        {
            src: 'https://cdn.jsdelivr.net/npm/node-forge/dist/forge.min.js',
            async: true,
            defer: true
        }
    ]
})
</script>

<template>
    <v-container>
        <v-file-input label="Choose file or drop here" @change="handleChange"></v-file-input>
        <v-textarea label="Text" v-model="text" @input="handleInput"></v-textarea>
        <v-text-field label="SHA384" v-model="hash"></v-text-field>
    </v-container>
</template>
