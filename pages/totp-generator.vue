<script setup>
import { ref } from 'vue'

const secret = ref(null)
const otp = ref(null)
const percent = ref(0)
const second = ref(0)
const copied = ref(false)
const history = ref([])
const headers = [
    { title: 'Secret', key: 'secret' },
    { title: 'CreatedAt', key: 'createdAt', value: item => new Date(item.createdAt).toLocaleString('sv') },
]

async function handleInput() {
    if (!secret.value) return
    const OTPAuth = await import('https://cdnjs.cloudflare.com/ajax/libs/otpauth/9.3.5/otpauth.esm.min.js')
    otp.value = new OTPAuth.TOTP({ secret: secret.value }).generate()
    history.value = [{ secret: secret.value, createdAt: new Date() }, ...history.value]
    localStorage.setItem('totp.history', JSON.stringify(history.value))
}

function copy() {
    const i = document.body.appendChild(document.createElement('input'))
    i.value = otp.value
    i.select()
    document.execCommand('copy')
    document.body.removeChild(i)
    copied.value = true
}

onMounted(() => {
    setInterval(() => {
        const sec = parseInt(Date.now() / 1000 % 30, 10)
        percent.value = parseInt(sec / 30 * 100, 10)
        second.value = sec
        if (sec === 0) handleInput()
    }, 1000)
    history.value = JSON.parse(localStorage.getItem('totp.history')) ?? []
})

async function handleClickRow(event, row) {
    secret.value = row.item.secret
    const OTPAuth = await import('https://cdnjs.cloudflare.com/ajax/libs/otpauth/9.3.5/otpauth.esm.min.js')
    otp.value = new OTPAuth.TOTP({ secret: secret.value }).generate()
}


useHead({
    title: 'TOTP Generator',
})
</script>

<template>
    <v-container>
        <v-text-field v-model="secret" label="SECRET KEY" width="220" @input="handleInput"></v-text-field>
        <div class="d-flex align-center">
            <v-text-field v-if="secret && otp" v-model="otp" width="220" hide-details
                class="flex-grow-0"></v-text-field>
            <v-btn v-if="secret && otp" @click="copy" color="secondary" class="mx-4">Copy</v-btn>
            <v-progress-circular v-if="secret && otp" color="primary" :model-value="percent" :size="60" :width="10">{{
                second
            }}</v-progress-circular>
        </div>
        <v-data-table v-if="history.length" :headers="headers" :items="history"
            @click:row="handleClickRow"></v-data-table>
        <v-snackbar v-model="copied">Copied!</v-snackbar>
    </v-container>
</template>
