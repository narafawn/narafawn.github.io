<script setup>
import { ref } from 'vue'

const user = ref('username@yahoo.co.jp')
const password = ref('password')
const host = ref('imap.mail.yahoo.co.jp')
const loggedIn = ref(false)
const dialog = ref(false)
const showCompose = ref(false)
const dialogTitle = ref('')
const dialogText = ref('')
const composeTo = ref('')
const composeSubject = ref('')
const composeBody = ref('')
const snackbar = ref(false)
const snackbarMessage = ref('')
const headers = [
    { title: 'Subject', key: 'parsed.subject' },
    { title: 'Sender', key: 'parsed.from.text' },
    { title: 'Date', key: 'parsed.date', value: item => new Date(item.parsed.date) }
]
const emails = ref([])

async function handleUserInput() {
    if (!user.value.includes('@')) return
    const hostname = user.value.split('@').at(-1)
    const hostnameMap = {
        'gmail.com': 'imap.gmail.com',
        'yahoo.co.jp': 'imap.mail.yahoo.co.jp',
        'ybb.ne.jp': 'imap.mail.yahoo.co.jp',
        'outlook.com': 'imap-mail.outlook.com',
        'live.com': 'imap-mail.outlook.com',
        'hotmail.com': 'imap-mail.outlook.com',
        'msn.com': 'imap-mail.outlook.com',
        'icloud.com': 'imap.mail.me.com',
        'biglobe.jp': 'mail.biglobe.ne.jp',
        'nifty.com': 'imap.nifty.com',
        '.*.ocn.ne.jp': 'imap.ocn.ne.jp',
        '.*.so-net.ne.jp': 'imap.so-net.ne.jp',
        '.*.eonet.ne.jp': 'imaps.eonet.ne.jp',
    }

    for (const hostnameRegex in hostnameMap) {
        if (new RegExp(hostnameRegex).test(hostname)) {
            host.value = hostnameMap[hostnameRegex]
            return
        }
    }

    const tlds = 'com,jp,net,xyz,org,info'.split(',')
    if (tlds.some(tld => new RegExp(`.+\.${tld}$`).test(hostname))) {
        const r = await fetch('https://ws.vercel.app/api/dns/ptr.js?' + new URLSearchParams({ hostname }))
        if (r.ok) {
            const imapHostnames = await r.json()
            if (imapHostnames.length) {
                host.value = imapHostnames[0]
            }
        }
    }
}

async function login() {
    const transport = {
        host: host.value,
        user: user.value,
        password: password.value,
        port: 993,
        tls: true,
    }
    const r = await fetch('https://ws.vercel.app/api/mail/imap.js', {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
        },
        body: JSON.stringify({ transport }),
    })
    if (r.ok) {
        loggedIn.value = true
        emails.value = (await r.json()).toReversed()
        localStorage.setItem('mail.host', host.value)
        localStorage.setItem('mail.user', user.value)
        localStorage.setItem('mail.password', password.value)
        composeTo.value = user.value
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

async function send() {
    const transport = {
        host: host.value.replace('imap', 'smtp'),
        port: 587,
        secure: false,
        auth: {
            user: user.value,
            pass: password.value,
        }
    }

    const mail = {
        from: user.value,
        to: composeTo.value,
        subject: composeSubject.value,
        text: composeBody.value
    }

    const r = await fetch('https://ep.vercel.app/api/mail/smtp.js', {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
        },
        body: JSON.stringify({ transport, mail }),
    })

    if (r.ok) {
        console.log(await r.json())
        showCompose.value = false
        login()
    }
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
    title: 'Web MAIL'
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
        <v-btn v-if="loggedIn" color="primary" @click="showCompose = true">Compose</v-btn>
        <v-btn v-if="loggedIn" color="secondary" @click="logout" class="ml-2">Logout</v-btn>
        <v-data-table v-if="loggedIn" :headers="headers" :items="emails" @click:row="handleClickRow"></v-data-table>
        <v-dialog v-model="dialog" width="auto">
            <v-card max-width="600" min-width="200" :title="dialogTitle">
                <v-card-text v-html="dialogText"></v-card-text>
                <template v-slot:actions>
                    <v-btn class="ms-auto" text="Close" @click="dialog = false"></v-btn>
                </template>
            </v-card>
        </v-dialog>
        <v-dialog v-model="showCompose" width="auto">
            <v-card width="400" :title="dialogTitle">
                <v-card-text>
                    <v-text-field label="To" v-model="composeTo"></v-text-field>
                    <v-text-field label="Subject" v-model="composeSubject"></v-text-field>
                    <v-textarea label="Body" v-model="composeBody"></v-textarea>
                </v-card-text>
                <template v-slot:actions>
                    <v-btn text="Send" color="primary" @click="send"></v-btn>
                    <v-btn text="Close" color="secondary" @click="showCompose = false"></v-btn>
                </template>
            </v-card>
        </v-dialog>
        <v-snackbar v-model="snackbar">{{ snackbarMessage }}</v-snackbar>
    </v-container>
</template>
