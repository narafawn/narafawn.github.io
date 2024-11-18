<script setup>
import { ref } from 'vue'

function generateUUID() {
    return ([1e7] + -1e3 + -4e3 + -8e3 + -1e11).replace(/[018]/g, c =>
        (c ^ crypto.getRandomValues(new Uint8Array(1))[0] & 15 >> c / 4).toString(16)
    );
}

const uuid = ref(generateUUID())

function generate() {
    uuid.value = generateUUID()
}

useHead({
    title: 'UUID Generator',
})
</script>

<template>
    <v-container>
        <v-btn @click="generate">Generate</v-btn>
        <v-text-field v-model="uuid" label="UUID" class="mt-8" style="max-width: 500px;"></v-text-field>
    </v-container>
</template>