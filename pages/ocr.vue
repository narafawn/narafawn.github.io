<script setup>
import { ref } from 'vue'

const content = ref(null)
const loading = ref(false)
const url = "https://script.google.com/macros/s/AKfycbzR9aMKMZxFR9ulFFuG1U6nFlyIHwswDFllg3iOhqnEA6i10Kl0/exec"

async function base64Image(file) {
    return new Promise(resolve => {
        const reader = new FileReader();
        reader.onload = () => resolve(reader.result.replace(/^.*,/, ""));
        reader.readAsDataURL(file);
    });
}

async function handleChange(e) {
    if (!e.target.files.length) return
    const file = e.target.files[0]
    loading.value = true
    content.value = await fetch(url, {
        method: "POST",
        body: await base64Image(file)
    }).then(r => r.text())
    loading.value = false
}

useHead({
    title: 'OCR',
})
</script>

<template>
    <v-container>
        <v-file-input label="Choose image file or drop here" @change="handleChange"></v-file-input>
        <v-progress-circular v-show="loading" :size="70" :width="7" color="purple" indeterminate></v-progress-circular>
        <v-textarea v-show="!loading" label="Content" v-model="content" rows="22"></v-textarea>
    </v-container>
</template>
