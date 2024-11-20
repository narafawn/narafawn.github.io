<script setup>
import { ref } from 'vue'

const input = ref('a:2:{i:0;s:12:"Sample array";i:1;a:2:{i:0;s:5:"Apple";i:1;s:6:"Orange";}}')
const output = ref('')

let php = null
async function handleInput() {
    output.value = ''
    const encoded = btoa(input.value)
    await php.run(`<?php var_export(unserialize(base64_decode('${encoded}')));`)
}

onMounted(async () => {
    const { PhpWeb } = await import('https://cdn.jsdelivr.net/npm/php-wasm/PhpWeb.mjs');
    php = new PhpWeb
    php.addEventListener('output', (event) => {
        output.value += event.detail;
    })
    handleInput()
})

useHead({
    title: 'unserialize.php',
})
</script>

<template>
    <v-container>
        <v-textarea label="Input" v-model="input" rows="5" @input="handleInput"></v-textarea>
        <v-textarea label="Output" v-model="output" rows="18"></v-textarea>
    </v-container>
</template>
