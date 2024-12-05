<script setup>
import { ref } from 'vue'

const ip = ref('')
const results = ref([{
    type: 'PTR',
    name: '8.8.8.8',
    content: 'dns.google',
    accessedAt: new Date().toLocaleString('sv')
}])

async function lookup() {
    if (!ip.value) return
    if (results.value[0].name === '8.8.8.8') results.value.splice(0, 1)
    const params = { ip: ip.value.trim() }
    const r = await fetch('https://fc.vercel.app/api/dns/reverse.js?' + new URLSearchParams(params))
    const contents = await r.json()
    console.log(contents)
    if (r.ok) {
        if (Array.isArray(contents)) {
            for (const content of contents) {
                if (Array.isArray(content)) {
                    for (const childContent of content) {
                        console.log('childContent')
                        results.value.unshift({
                            type: 'PTR',
                            name: ip.value,
                            content: childContent,
                            accessedAt: new Date().toLocaleString('sv')
                        })
                    }
                } else {
                    results.value.unshift({
                        type: 'PTR',
                        name: ip.value,
                        content,
                        accessedAt: new Date().toLocaleString('sv')
                    })
                }
            }
        } else {
            results.value.unshift({
                type: 'PTR',
                name: ip.value,
                content: contents,
                accessedAt: new Date().toLocaleString('sv')
            })
        }
    } else {
        results.value.unshift({
            type: 'PTR',
            name: ip.value,
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
    return { class: typeof item.item.content === 'string' && item.item.content.startsWith('getHostByAddr E') ? 'unavailable-row' : 'available-row' }
}

let intervalId
onMounted(() => {
    intervalId = setInterval(lookup, 30000)
})

onUnmounted(() => clearInterval(intervalId))

useHead({
    title: 'Reverse IP Lookup Checker',
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
            <v-text-field label="Domain" placeholder="example.com" v-model="ip" class="flex-grow-0 mr-2" hide-details
                width="350" @input="handleInput" @keyup.enter="lookup" />
            <v-btn @click="lookup" color="primary">Lookup</v-btn>
        </div>
        <v-data-table :items="results" :row-props="getRowClass"></v-data-table>
    </v-container>
</template>
