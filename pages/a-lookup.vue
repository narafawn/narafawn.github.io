<script setup>
import { ref } from 'vue'

const hostname = ref('')
const results = ref([{
    type: 'A',
    name: 'example.com',
    content: '127.0.0.1',
    accessedAt: new Date().toLocaleString('sv')
}])

async function lookup() {
    if (!hostname.value) return
    if (results.value[0].name === 'example.com') results.value.splice(0, 1)
    const params = { hostname: hostname.value.trim(), rrtype: 'A' }
    const r = await fetch('https://ws.vercel.app/api/dns/dns.js?' + new URLSearchParams(params))
    const contents = await r.json()
    console.log(contents)
    if (r.ok) {
        if (Array.isArray(contents)) {
            for (const content of contents) {
                if (Array.isArray(content)) {
                    for (const childContent of content) {
                        console.log('childContent')
                        results.value.unshift({
                            type: 'A',
                            name: hostname.value,
                            content: childContent,
                            accessedAt: new Date().toLocaleString('sv')
                        })
                    }
                } else {
                    results.value.unshift({
                        type: 'A',
                        name: hostname.value,
                        content,
                        accessedAt: new Date().toLocaleString('sv')
                    })
                }
            }
        } else {
            results.value.unshift({
                type: 'A',
                name: hostname.value,
                content: contents,
                accessedAt: new Date().toLocaleString('sv')
            })
        }
    } else {
        results.value.unshift({
            type: 'A',
            name: hostname.value,
            content: contents,
            accessedAt: new Date().toLocaleString('sv')
        })
    }
}

function debounce(func, delay) {
    let id
    return function (...args) {
        if (id) {
            clearTimeout(id)
        }

        id = setTimeout(() => {
            func.apply(this, args)
        }, delay)
    };
}

const handleInput = debounce(lookup, 1000)

function getRowClass(item) {
    return { class: typeof item.item.content === 'string' && item.item.content.startsWith('queryA E') ? 'unavailable-row' : 'available-row' }
}

onMounted(() => {
    setInterval(lookup, 30000)
})

useHead({
    title: 'A Lookup Checker',
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
        <div class="d-flex align-center">
            <v-text-field label="Domain" placeholder="example.com" v-model="hostname" class="flex-grow-0 mr-2"
                hide-details width="350" @input="handleInput" @keyup.enter="lookup" />
            <v-btn @click="lookup" color="primary">Lookup</v-btn>
        </div>
        <v-data-table :items="results" :row-props="getRowClass"></v-data-table>
    </v-container>
</template>
