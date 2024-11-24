<script setup>
import { ref } from 'vue'

const q = ref('漢字かな交じり文にふりがなを振ること。')
const html = ref('<ruby>漢字<rt>かんじ</rt></ruby>かな<ruby>交<rt>ま</rt></ruby>じり<ruby>文<rt>ぶん</rt></ruby>にふりがなを<ruby>振<rt>ふ</rt></ruby>ること。')
const showSnackbar = ref(false)
const snackbarMessage = ref('')

function generateRubyHtml(apiResponse) {
    const words = apiResponse.result.word

    let html = ''
    words.forEach(word => {
        if (word.subword) {
            word.subword.forEach(sub => {
                if (sub.furigana !== sub.surface && !/^[ァ-ヶー]+$/.test(sub.surface)) {
                    html += `<ruby>${sub.surface}<rt>${sub.furigana}</rt></ruby>`
                } else {
                    html += sub.surface
                }
            })
        } else if (word.furigana) {
            html += `<ruby>${word.surface}<rt>${word.furigana}</rt></ruby>`
        } else {
            html += word.surface
        }
    })

    return html.replaceAll('\n', '<br>')
}

const invalidCharactors = [
    { char: '·', replacement: 'EQXEN1MZM0MBU769' },
    { char: '–', replacement: 'D8RD70IFD0AIH6X8' },
]

async function generate() {
    if (!q.value) {
        return
    }

    let query = q.value
    for (const { char, replacement } of invalidCharactors) {
        query = query.replaceAll(char, replacement)
    }

    html.value = ''
    const appid = 'dj00aiZpPXYwNnZyWW9hU3lOYyZzPWNvbnN1bWVyc2VjcmV0Jng9YTE-'
    const response = await fetch('https://jlp.yahooapis.jp/FuriganaService/V2/furigana?' + new URLSearchParams({ appid }), {
        method: 'POST',
        body: JSON.stringify({
            id: Math.random().toString(),
            jsonrpc: '2.0',
            method: 'jlp.furiganaservice.furigana',
            params: {
                q: query,
                grade: 1
            }
        })
    })
    const responseData = await response.json()
    console.log(responseData)
    if (responseData.error) {
        snackbarMessage.value = responseData.error.message
        showSnackbar.value = true
        return
    }
    let htmlValue = generateRubyHtml(responseData)
    for (const { char, replacement } of invalidCharactors) {
        htmlValue = htmlValue.replaceAll(replacement, char)
    }
    html.value = htmlValue
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

const debouncedGenerate = debounce(generate, 1000)

useHead({
    title: 'ふりがな（ルビ）'
})
</script>

<style>
.output {
    font-size: 1.3rem;
}

rt {
    color: blue;
    font-size: 0.7rem;
}
</style>

<template>
    <v-container>
        <p class="mb-2">漢字かな交じり文に、ひらがなとローマ字のふりがな（ルビ）を付けます。</p>
        <v-textarea label="テキスト" v-model="q" @input="debouncedGenerate"></v-textarea>
        <div v-html="html" class="output"></div>
        <a href="https://developer.yahoo.co.jp/webapi/jlp/furigana/v2/furigana.html" target="_blank"
            class="float-right">Web Services by
            Yahoo! JAPAN</a>
        <v-snackbar v-model="showSnackbar">{{ snackbarMessage }}</v-snackbar>
    </v-container>
</template>
