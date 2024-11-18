<script setup>
import { ref } from 'vue'

const secret = ref(null)
const otp = ref(null)
const percent = ref(0)
const second = ref(0)
async function handleInput() {
    if (!secret.value) return
    const OTPAuth = await import('https://cdnjs.cloudflare.com/ajax/libs/otpauth/9.3.5/otpauth.esm.min.js')
    otp.value = new OTPAuth.TOTP({ secret: secret.value }).generate()
}


onMounted(() => {
    setInterval(() => {
        const sec = parseInt(Date.now() / 1000 % 30, 10)
        percent.value = parseInt(sec / 30 * 100, 10)
        second.value = sec
        if (sec === 0) handleInput()
    }, 1000)
})


useHead({
    title: 'TOTP Generator',
})
</script>

<template>
    <v-container>
        <v-text-field v-model="secret" label="SECRET KEY" width="200px" @input="handleInput"></v-text-field>
        <v-text-field v-if="secret && otp" v-model="otp" width="200px"></v-text-field>
        <v-progress-circular v-if="secret && otp" color="primary" :model-value="percent" :size="60" :width="10">{{
            second
            }}</v-progress-circular>
    </v-container>
</template>
