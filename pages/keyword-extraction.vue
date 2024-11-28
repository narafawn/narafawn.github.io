<script setup>
import { ref } from 'vue'

const query = ref('東京ミッドタウンから国立新美術館まで歩いて5分で着きます。')
const phrases = ref([
    {
        score: 100,
        text: "東京ミッドタウン"
    },
    {
        score: 73,
        text: "国立新美術館"
    },
    {
        score: 37,
        text: "5分"
    }
])
const showSnackbar = ref(false)
const snackbarMessage = ref('')

async function extract() {
    let q = query.value
    if (!q) return

    phrases.value = []
    const appid = 'dj00aiZpPXYwNnZyWW9hU3lOYyZzPWNvbnN1bWVyc2VjcmV0Jng9YTE-'
    const response = await fetch('https://jlp.yahooapis.jp/KeyphraseService/V2/extract?' + new URLSearchParams({ appid }), {
        method: 'POST',
        body: JSON.stringify({
            id: Math.random().toString(),
            jsonrpc: '2.0',
            method: 'jlp.keyphraseservice.extract',
            params: {
                q
            }
        })
    })
    const obj = await response.json()
    console.log(obj)
    if (obj.error) {
        snackbarMessage.value = obj.error.message
        showSnackbar.value = true
        return
    }
    phrases.value = obj.result.phrases
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
    }
}

const debouncedExtract = debounce(extract, 1000)

useHead({
    title: 'キーフレーズ抽出'
})
</script>
<template>
    <v-container>
        <v-textarea label="テキスト" v-model="query" @input="debouncedExtract"></v-textarea>
        <v-data-table :items="phrases"></v-data-table>
        <a href="https://developer.yahoo.co.jp/webapi/jlp/keyphrase/v2/extract.html" target="_blank"
            class="float-right">Web
            Services by
            Yahoo! JAPAN</a>
        <v-snackbar v-model="showSnackbar">{{ snackbarMessage }}</v-snackbar>
    </v-container>
</template>
