<script setup>
import { ref } from 'vue'

const content = ref(null)

async function handleChange(e) {
    if (!e.target.files.length) return
    const file = e.target.files[0]
    content.value = await new Response(file.stream().pipeThrough(new DecompressionStream("gzip"))).text()
}

useHead({
    title: 'file.gz Extractor',
})
</script>

<template>
    <v-container>
        <v-file-input label="Choose .gz file or drop here" @change="handleChange"></v-file-input>
        <v-textarea label="Content" v-model="content" rows="20"></v-textarea>
    </v-container>
</template>
