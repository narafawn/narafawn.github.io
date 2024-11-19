<script setup>
import { ref } from 'vue'

const text = ref('')
const count = ref(null)
const ignoreEmpty = ref(false)

async function handleInput() {
    let lines = text.value.split('\n')
    if (ignoreEmpty.value) {
        lines = lines.filter(line => line.trim())
    }
    count.value = lines.length
}

useHead({
    title: 'Count Lines',
})
</script>

<template>
    <v-container>
        <v-textarea label="Text" v-model="text" rows="20" @input="handleInput"></v-textarea>
        <div class="d-flex">
            <v-text-field v-model="count" label="Count" class="flex-grow-0" width="200px"></v-text-field>
            <v-checkbox label="Ignore empty line" v-model="ignoreEmpty" @change="handleInput"></v-checkbox>
        </div>
    </v-container>
</template>
