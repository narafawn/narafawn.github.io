<script setup>
import { ref } from 'vue'

const input = ref(`const params = {
  "foo": 100,
  "_foo": true,
  "$foo": "bar",
  "キー": null
}`)
const output = ref('')
const copied = ref(false)

async function handleInput() {
    // \u4E00-\u9FFF 漢字
    // \u3040-\u309F ひらがな
    // \u30A0-\u30FF 全角カタカナ
    // \uFF65-\uFF9F 半角カタカナ
    // \w A-Za-z0-9_
    // $ $
    // \1 Single quote or Double quote
    // \s White space character
    output.value = input.value.replaceAll(/(['"])([\u4E00-\u9FFF\u3040-\u309F\u30A0-\u30FF\uFF65-\uFF9F\w$]+?)\1\s*:/g, '$2:')
}

onMounted(handleInput)

function copy() {
    const i = document.body.appendChild(document.createElement('textarea'))
    i.value = output.value
    i.select()
    document.execCommand('copy')
    document.body.removeChild(i)
    copied.value = true
}

useHead({
    title: 'Remove JavaScript key quotes',
})
</script>

<template>
    <v-container>
        <v-textarea label="Input" v-model="input" rows="10" @input="handleInput"></v-textarea>
        <v-textarea label="Output" v-model="output" rows="10" readonly></v-textarea>
        <v-btn @click="copy" color="secondary">Copy</v-btn>
        <v-snackbar v-model="copied">Copied!</v-snackbar>
    </v-container>
</template>
