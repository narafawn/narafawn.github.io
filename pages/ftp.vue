<script setup>
import { ref } from 'vue'

const user = ref('username@ftp.example.com')
const password = ref('password')
const host = ref('ftp.example.com')
const loggedIn = ref(false)
const snackbar = ref(false)
const snackbarMessage = ref('')
const headers = [
    { title: 'Name', key: 'name' },
    { title: 'Size', key: 'size', value: item => formatBytes(item.size), align: 'end' },
    { title: 'ModifiedAt', key: 'modifiedAt', value: item => formatDate(new Date(item.modifiedAt)) },
]
const files = ref([])

function formatBytes(bytes, decimals = 1) {
    if (bytes === 0) return '0  B '
    const k = 1024 // 1KB = 1024 Bytes
    const sizes = ['  B ', ' KB', ' MB', ' GB', ' TB', ' PB']
    const i = Math.floor(Math.log(bytes) / Math.log(k))
    return parseFloat((bytes / Math.pow(k, i)).toFixed(decimals)) + sizes[i]
}

function formatDate(date) {
    const diff = new Date() - date
    return diff / 86400000 < 1 ? date.toLocaleTimeString('sv') : date.toLocaleDateString('sv')
}

async function handleUserInput() {
    if (!user.value.includes('@')) return
    const hostname = user.value.split('@').at(-1)
    const tlds = 'com,jp,net,xyz,org,info'.split(',')
    if (tlds.some(tld => new RegExp(`.+\.${tld}$`).test(hostname))) {
        const r = await fetch('https://ep.vercel.app/api/dns/ptr.js?' + new URLSearchParams({ hostname }))
        if (r.ok) {
            const imapHostnames = await r.json()
            if (imapHostnames.length) {
                host.value = imapHostnames[0]
            }
        }
    }
}

async function login() {
    const options = {
        host: host.value,
        user: user.value,
        password: password.value
    }
    const r = await fetch('https://ws.vercel.app/api/ftp/list.js?' + new URLSearchParams(options), { method: "POST" })
    if (r.ok) {
        loggedIn.value = true
        files.value = await r.json()
        localStorage.setItem('ftp.host', host.value)
        localStorage.setItem('ftp.user', user.value)
        localStorage.setItem('ftp.password', password.value)
    } else {
        logout()
        snackbarMessage.value = await r.json()
        snackbar.value = true
    }
}

function logout() {
    localStorage.removeItem('ftp.host')
    localStorage.removeItem('ftp.user')
    localStorage.removeItem('ftp.password')
    loggedIn.value = false
}

function handleClickRow(event, row) {
    if (row.item.size / 1024 / 1024 > 32) {
        snackbarMessage.value = 'Sorry Cannot download a large file.😭'
        snackbar.value = true
        return
    }
    const options = {
        host: host.value,
        user: user.value,
        password: password.value,
        filename: row.item.name
    }
    location = 'https://ws.vercel.app/api/ftp/download.js?' + new URLSearchParams(options)
}

async function handleFileChange(e) {
    if (!e.target.files.length) return

    const body = new FormData()
    for (const file of e.target.files) {
        body.append('files', file)
    }

    const options = {
        host: host.value,
        user: user.value,
        password: password.value
    }
    const r = await fetch('https://ws.vercel.app/api/ftp/upload.js?' + new URLSearchParams(options), { method: "POST", body })
    if (r.ok) {
        snackbarMessage.value = (await r.json()).message
        snackbar.value = true
    } else {
        snackbarMessage.value = await r.json()
        snackbar.value = true
    }
}

onMounted(() => {
    const savedHost = localStorage.getItem('ftp.host')
    const savedUser = localStorage.getItem('ftp.user')
    const savedPassword = localStorage.getItem('ftp.password')
    if (savedHost && savedUser && savedPassword) {
        host.value = savedHost
        user.value = savedUser
        password.value = savedPassword
        login()
    }
})

useHead({
    title: 'Web FTP'
})
</script>

<template>
    <v-container>
        <v-form v-if="!loggedIn" @submit.prevent="login">
            <v-text-field v-model="user" label="User" @input="handleUserInput"></v-text-field>
            <v-text-field v-model="password" label="Password"></v-text-field>
            <v-text-field v-model="host" label="Host"></v-text-field>
            <v-btn color="primary" type="submit">Login</v-btn>
        </v-form>
        <div class="d-flex align-center">
            <v-file-input v-if="loggedIn" label="Choose file to upload or drop here" @change="handleFileChange"
                hide-details multiple></v-file-input>
            <v-btn v-if="loggedIn" color="secondary" @click="logout" class="ml-2">Logout</v-btn>
        </div>
        <v-data-table v-if="loggedIn" :headers="headers" :items="files" @click:row="handleClickRow"></v-data-table>
        <v-snackbar v-model="snackbar">{{ snackbarMessage }}</v-snackbar>
    </v-container>
</template>
