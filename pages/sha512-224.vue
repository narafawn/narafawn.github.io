<script setup>
import { ref } from 'vue'

const text = ref('')
const hash = ref('6ed0dd02806fa89e25de060c19d3ac86cabb87d6a0ddd05c333b84f4')

function doHash(string) {
    hash.value = forge.md.sha512.sha224.create().update(string).digest().toHex()
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
    title: 'SHA512/224 Hash',
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
        <v-text-field label="SHA512/224" v-model="hash"></v-text-field>
    </v-container>
</template>
