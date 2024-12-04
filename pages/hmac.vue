<script setup>
import { ref } from 'vue'


const algorithm = ref('SHA256')
const functions = [
    'MD5',
    'SHA1',
    'SHA256',
    'SHA384',
    'SHA512',
    'SHA512/256',
    'SHA512/224'
]
const key = ref('key')
const text = ref('The quick brown fox jumps over the lazy dog')
const hash = ref('f7bc83f430538424b13298e6aa6fb143ef4d59a14946175997479dbc2d1a3cd8')
let lastHashedString = 'The quick brown fox jumps over the lazy dog'

function doHash(string) {
    lastHashedString = string
    const hmac = forge.hmac.create()
    hmac.start(algorithm.value.toLowerCase(), key.value)
    hmac.update(string)
    hash.value = hmac.digest().toHex()
}

function redoHash() {
    hash.value = ''
    doHash(lastHashedString)
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
    title: 'HMAC Hash',
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
        <v-select label="Hash Function" v-model="algorithm" :items="functions" @update:modelValue="redoHash"></v-select>
        <v-textarea label="Key" v-model="key" @input="redoHash"></v-textarea>
        <v-file-input label="Choose file or drop here" @change="handleChange"></v-file-input>
        <v-textarea label="Text" v-model="text" @input="handleInput"></v-textarea>
        <v-text-field label="HMAC" v-model="hash"></v-text-field>
    </v-container>
</template>
