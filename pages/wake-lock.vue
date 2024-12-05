<script setup>
import { ref } from 'vue'

const message = ref('Wake lock is not supported by this browser.')
const elapsed = ref(0)
let start = 0
let intervalId

async function requestWakeLock() {
    try {
        const wakeLock = await navigator.wakeLock.request('screen')
        message.value = 'Acquired'
        wakeLock.addEventListener('release', () => {
            console.log('Released')
            message.value = 'Released'
        })
        if (intervalId) clearInterval(intervalId)
        start = Date.now()
        intervalId = setInterval(() => {
            const s = Math.floor((Date.now() - start) / 1000)
            elapsed.value = `${Math.floor(s / 60).toString().padStart(2, 0)}:${(s % 60).toString().padStart(2, 0)}`
        }, 1000)
    } catch (error) {
        message.value = `${error.name}, ${error.message}`
    }
}

onMounted(async () => {
    requestWakeLock()
    document.addEventListener('visibilitychange', () => document.visibilityState === 'visible' && requestWakeLock())
})

onUnmounted(() => {
    console.log('onUnmounted')
    if (intervalId) clearInterval(intervalId)
})

useHead({
    title: 'Screen Wake Lock',
})
</script>

<template>
    <v-container>
        {{ message }}
        <a href="https://developer.mozilla.org/en-US/docs/Web/API/Screen_Wake_Lock_API" target="_blank">About Screen
            Wake Lock</a>
        <h1 v-if="message === 'Acquired'">{{ elapsed }}</h1>
    </v-container>
</template>
