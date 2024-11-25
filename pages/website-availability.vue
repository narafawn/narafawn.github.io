<script setup>
import { ref } from 'vue'

const urls = ref(`https://example.com/
https://foobar.com
http://httpbin.org/status/404`)
const results = ref([
    {
        url: 'https://example.com/', status: '200', obj: {
            "ok": true,
            "headers": {
                "accept-ranges": "bytes",
                "age": "319703",
                "cache-control": "max-age=604800",
                "content-type": "text/html; charset=UTF-8",
                "date": "Mon, 25 Nov 2024 11:30:58 GMT",
                "etag": "\"3147526947+gzip\"",
                "expires": "Mon, 02 Dec 2024 11:30:58 GMT",
                "last-modified": "Thu, 17 Oct 2019 07:18:26 GMT",
                "server": "ECAcc (sed/58ED)",
                "vary": "Accept-Encoding",
                "x-cache": "HIT"
            },
            "redirected": false,
            "status": 200,
            "statusText": "OK",
            "type": "basic",
            "url": "https://example.com/",
            "bodyUsed": false
        }
    },
    {
        url: 'https://foobar.com', status: 'error sending request for url (https://foobar.com/): client error (Connect): invalid peer certificate: Expired', obj: {
            "name": "TypeError",
            "message": "error sending request for url (https://foobar.com/): client error (Connect): invalid peer certificate: Expired"
        }
    },
    {
        url: 'http://httpbin.org/status/404', status: '404', obj: {
            "ok": false,
            "headers": {
                "access-control-allow-credentials": "true",
                "access-control-allow-origin": "*",
                "connection": "keep-alive",
                "content-length": "0",
                "content-type": "text/html; charset=utf-8",
                "date": "Mon, 25 Nov 2024 11:31:35 GMT",
                "server": "gunicorn/19.9.0"
            },
            "redirected": false,
            "status": 404,
            "statusText": "Not Found",
            "type": "basic",
            "url": "http://httpbin.org/status/404",
            "bodyUsed": false
        }
    },
])
const dialog = ref(false)
const dialogText = ref('')
const headers = [
    { title: 'URL', key: 'url' },
    { title: 'Status', key: 'status' },
]

async function check() {
    results.value = []
    const target_urls = urls.value.split('\n').filter(l => l.startsWith('http')).slice(0, 100)
    for (const url of target_urls) {
        fetch('https://wsa.deno.dev?' + new URLSearchParams({ url }))
            .then(async r => {
                const obj = await r.json()
                const index = results.value.findIndex(i => i.url === url)
                results.value[index].status = r.ok ? obj.status : obj.message
                results.value[index].obj = obj
            })
        results.value.push({ url })
    }
}

function getRowClass(item) {
    if (/^\d{3}$/.test(item.item.status)) {
        const status = parseInt(item.item.status, 10)
        if (status >= 200 && status <= 299) return { class: 'available-row' }
        else return { class: 'unavailable-row' }
    }
}

function handleClickRow(event, row) {
    if (!row.item.obj) return
    dialogText.value = JSON.stringify(row.item.obj, null, 2)
    dialog.value = true
}


useHead({
    title: 'Website Availability',
})
</script>
<style>
.available-row {
    background: #C8E6C9;
}

.unavailable-row {
    background: #FFCDD2;
}
</style>
<template>
    <v-container>
        <v-textarea label="URLs" v-model="urls"></v-textarea>
        <v-btn color="primary" @click="check">Check Availability</v-btn>
        <v-data-table :headers="headers" :items="results" :row-props="getRowClass"
            @click:row="handleClickRow"></v-data-table>
        <v-dialog v-model="dialog" width="auto" scrollable>
            <v-card>
                <template v-slot:actions>
                    <v-btn class="ms-auto" text="Ok" @click="dialog = false"></v-btn>
                </template>
                <v-card-text>
                    <pre><code>{{ dialogText }}</code></pre>
                </v-card-text>
            </v-card>
        </v-dialog>
    </v-container>
</template>
