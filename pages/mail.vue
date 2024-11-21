<script setup>
import { ref } from 'vue'

const host = ref('imap.example.com')
const user = ref('username@example.com')
const password = ref('password')
const loggedIn = ref(false)
const dialog = ref(false)
const dialogTitle = ref('')
const dialogText = ref('')
const snackbar = ref(false)
const snackbarMessage = ref('')
const headers = [
    { title: 'Subject', key: 'parsed.subject' },
    { title: 'Sender', key: 'parsed.from.text' },
    { title: 'Date', key: 'parsed.date', value: item => new Date(item.parsed.date) }
]
const emails = ref([])

async function login() {
    const params = { host: host.value, user: user.value, password: password.value }
    const r = await fetch('https://ws.vercel.app/api/imap.js?' + new URLSearchParams(params))
    if (r.ok) {
        loggedIn.value = true
        emails.value = await r.json()
        localStorage.setItem('mail.host', host.value)
        localStorage.setItem('mail.user', user.value)
        localStorage.setItem('mail.password', password.value)
    } else {
        logout()
        snackbarMessage.value = await r.json()
        snackbar.value = true
    }
}

function logout() {
    localStorage.removeItem('mail.host')
    localStorage.removeItem('mail.user')
    localStorage.removeItem('mail.password')
    loggedIn.value = false
}

function handleClickRow(event, row) {
    dialogTitle.value = row.item.parsed.subject
    dialogText.value = row.item.parsed.text.replaceAll('\n', '<br>')
    dialog.value = true
}

onMounted(() => {
    const savedHost = localStorage.getItem('mail.host')
    const savedUser = localStorage.getItem('mail.user')
    const savedPassword = localStorage.getItem('mail.password')
    if (savedHost && savedUser && savedPassword) {
        host.value = savedHost
        user.value = savedUser
        password.value = savedPassword
        login()
    }
})

useHead({
    title: 'MAIL'
})
</script>

<template>
    <v-container>
        <v-form v-if="!loggedIn" @submit.prevent="login">
            <v-text-field v-model="host" label="Host"></v-text-field>
            <v-text-field v-model="user" label="User"></v-text-field>
            <v-text-field v-model="password" label="Password"></v-text-field>
            <v-btn color="primary" type="submit">Login</v-btn>
        </v-form>
        <v-btn v-if="loggedIn" color="primary" @click="logout">Compose</v-btn>
        <v-btn v-if="loggedIn" color="secondary" @click="logout">Logout</v-btn>
        <v-data-table v-if="loggedIn" :headers="headers" :items="emails" @click:row="handleClickRow"></v-data-table>
        <v-dialog v-model="dialog" width="auto">
            <v-card max-width="400" min-width="200" :title="dialogTitle">
                <v-card-text v-html="dialogText"></v-card-text>
                <template v-slot:actions>
                    <v-btn class="ms-auto" text="Close" @click="dialog = false"></v-btn>
                </template>
            </v-card>
        </v-dialog>
        <v-snackbar v-model="snackbar">{{ snackbarMessage }}</v-snackbar>
    </v-container>
</template>