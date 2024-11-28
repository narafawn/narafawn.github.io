<script setup>
import { ref } from 'vue'

const query = ref('美しい水車小屋の娘')
const tokens = ref([
    [
        "美しい",
        "うつくしい",
        "美しい",
        "形容詞",
        "*",
        "イ形容詞イ段",
        "基本形"
    ],
    [
        "水車",
        "すいしゃ",
        "水車",
        "名詞",
        "普通名詞",
        "*",
        "*"
    ],
    [
        "小屋",
        "こや",
        "小屋",
        "名詞",
        "普通名詞",
        "*",
        "*"
    ],
    [
        "の",
        "の",
        "の",
        "助詞",
        "接続助詞",
        "*",
        "*"
    ],
    [
        "娘",
        "むすめ",
        "娘",
        "名詞",
        "普通名詞",
        "*",
        "*"
    ]
])
const showSnackbar = ref(false)
const snackbarMessage = ref('')
const headers = [
    { title: '表記', key: '0' },
    { title: '読みがな', key: '1' },
    { title: '基本形表記', key: '2' },
    { title: '品詞', key: '3' },
    { title: '品詞細分類', key: '4' },
    { title: '活用型', key: '5' },
    { title: '活用形', key: '6' },
]

async function parse() {
    let q = query.value
    if (!q) return

    tokens.value = []
    const appid = 'dj00aiZpPXYwNnZyWW9hU3lOYyZzPWNvbnN1bWVyc2VjcmV0Jng9YTE-'
    const response = await fetch('https://jlp.yahooapis.jp/MAService/V2/parse?' + new URLSearchParams({ appid }), {
        method: 'POST',
        body: JSON.stringify({
            id: Math.random().toString(),
            jsonrpc: '2.0',
            method: 'jlp.maservice.parse',
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
    tokens.value = obj.result.tokens
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

const debouncedParse = debounce(parse, 1000)

useHead({
    title: '形態素解析'
})
</script>
<template>
    <v-container>
        <v-textarea label="テキスト" v-model="query" @input="debouncedParse"></v-textarea>
        <v-data-table :headers="headers" :items="tokens"></v-data-table>
        <a href="https://developer.yahoo.co.jp/webapi/jlp/ma/v2/parse.html" target="_blank" class="float-right">Web
            Services by
            Yahoo! JAPAN</a>
        <v-snackbar v-model="showSnackbar">{{ snackbarMessage }}</v-snackbar>
    </v-container>
</template>
