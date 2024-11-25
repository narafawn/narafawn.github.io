<script setup>
import { ref } from 'vue'

const q = ref('漢字かな交じり文にふりがなを振ること。')
const correction = ref('杯 さかずき\n僕 しもべ\n司 し')
const html = ref('<ruby>漢字<rt>かんじ</rt></ruby>かな<ruby>交<rt>ま</rt></ruby>じり<ruby>文<rt>ぶん</rt></ruby>にふりがなを<ruby>振<rt>ふ</rt></ruby>ること。')
const showSnackbar = ref(false)
const snackbarMessage = ref('')
let responseData = { result: { word: [] } }

function generateRubyHtml(apiResponse) {
    const correctionMap = Object.fromEntries(correction.value.split('\n').map(l => l.split(/\s/)).filter(l => l.length === 2))
    const words = apiResponse.result.word

    let html = ''
    words.forEach(word => {
        if (word.subword) {
            word.subword.forEach(sub => {
                if (sub.furigana !== sub.surface && !/^[ァ-ヶー]+$/.test(sub.surface)) {
                    const furigana = correctionMap[sub.surface] ?? sub.furigana
                    html += `<ruby>${sub.surface}<rt>${furigana}</rt></ruby>`
                } else {
                    html += sub.surface
                }
            })
        } else if (word.furigana) {
            const furigana = correctionMap[word.surface] ?? word.furigana
            html += `<ruby>${word.surface}<rt>${furigana}</rt></ruby>`
        } else {
            html += word.surface
        }
    })

    return html.replaceAll('\n', '<br>')
}

const invalidCharactors = [
    { char: '·', replacement: 'EQXEN1MZM0MBU769' },
    { char: '–', replacement: 'D8RD70IFD0AIH6X8' },
    { char: '—', replacement: 'UJF3VQEU1MNWHNB2' },
    { char: '≥', replacement: 'ZMNCMYIGSSCFWB6J' },
    { char: '≤', replacement: 'A0GIEVZ0J1NY31TQ' },
    { char: '↔', replacement: 'A90NN8LY52EO64W4' },
    { char: '~', replacement: 'QFNBOA3U2EV6UJ37' },
    { char: '∑', replacement: 'ZBDVSMMEJMQRXSC3' },
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
    responseData = await response.json()
    console.log(responseData)
    correct()
}

function correct(e) {
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
        <div class="d-flex">
            <p class="mb-2">漢字かな交じり文に、ひらがなとローマ字のふりがな（ルビ）を付けます。</p>
            <details>
                <summary>補正ふりがな</summary>
                補正したい漢字を右クリックし、検証を押し、&lt;ruby&gt;漢字&lt;rt&gt;かんじ&lt;/rt&gt;&lt;/ruby&gt;の漢字を左にふりがなを右に入力してください。
                <v-textarea label="補正ふりがな" v-model="correction" @input="correctionInput"></v-textarea>
            </details>
        </div>
        <v-textarea label="テキスト" v-model="q" @input="debouncedGenerate"></v-textarea>
        <div v-html="html" class="output"></div>
        <a href="https://developer.yahoo.co.jp/webapi/jlp/furigana/v2/furigana.html" target="_blank"
            class="float-right">Web Services by
            Yahoo! JAPAN</a>
        <v-snackbar v-model="showSnackbar">{{ snackbarMessage }}</v-snackbar>
    </v-container>
</template>
