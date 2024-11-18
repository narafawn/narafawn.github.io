<script setup>
import { ref } from 'vue'

const password = ref(Math.random().toString(16).slice(-8))

function generate() {
    password.value = Math.random().toString(16).slice(-8)
}

useHead({
    title: 'Password Generator',
})
</script>

<template>
    <v-container>
        <v-btn @click="generate">Generate</v-btn>
        <v-text-field v-model="password" label="Password" class="mt-8" width="200px"></v-text-field>
    </v-container>
</template>