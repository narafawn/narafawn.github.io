<script setup>
import { ref } from 'vue'

const message = ref('Wake lock is not supported by this browser.')

async function requestWakeLock() {
    try {
        const wakeLock = await navigator.wakeLock.request('screen')
        message.value = 'Acquired'
        wakeLock.addEventListener('release', () => {
            console.log('Released')
            message.value = 'Released'
        })
    } catch (error) {
        message.value = `${error.name}, ${error.message}`
    }
}

onMounted(async () => {
    requestWakeLock()
    document.addEventListener('visibilitychange', () => document.visibilityState === 'visible' && requestWakeLock())
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
    </v-container>
</template>
