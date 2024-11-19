<script setup>
import { ref } from 'vue'

const input = ref('')
const output = ref('')
const pattern = ref('')
const replacement = ref('')
const useRegex = ref(false)
const ignoreCase = ref(false)
const multiline = ref(false)

async function handleInput() {
    let patternValue = pattern.value
    if (useRegex.value) {
        let flags = 'g'
        if (ignoreCase.value) {
            flags += 'i'
        }
        if (multiline.value) {
            flags += 'm'
        }
        patternValue = new RegExp(pattern.value, flags)
    }
    output.value = input.value.replaceAll(patternValue, replacement.value)
}

useHead({
    title: 'Replace text',
})
</script>

<template>
    <v-container>
        <v-textarea label="Input" v-model="input" rows="10" @input="handleInput"></v-textarea>
        <div class="d-flex">
            <v-text-field v-model="pattern" label="Pattern" class="flex-grow-0" width="200px"
                @input="handleInput"></v-text-field>
            <v-text-field v-model="replacement" label="Replacement" class="flex-grow-0 ml-4" width="200px"
                @input="handleInput"></v-text-field>
            <v-checkbox label="Use Regex" v-model="useRegex" @change="handleInput"></v-checkbox>
            <v-checkbox v-if="useRegex" label="Ignore Case" v-model="ignoreCase" @change="handleInput"></v-checkbox>
            <v-checkbox v-if="useRegex" label="Multiline" v-model="multiline" @change="handleInput"></v-checkbox>
        </div>
        <v-textarea label="Output" v-model="output" rows="10"></v-textarea>
    </v-container>
</template>
