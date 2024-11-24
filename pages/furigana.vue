<script setup>
import { ref } from 'vue'

const q = ref(`漢字かな交じり文にふりがなを振ること。`)
const html = ref('<ruby>漢字<rt>かんじ</rt></ruby>かな<ruby>交<rt>ま</rt></ruby>じり<ruby>文<rt>ぶん</rt></ruby>にふりがなを<ruby>振<rt>ふ</rt></ruby>ること。')

function generateRubyHtml(apiResponse) {
    const words = apiResponse.result.word

    let html = ''
    words.forEach(word => {
        if (word.subword) {
            word.subword.forEach(sub => {
                if (sub.furigana !== sub.surface) {
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

async function generate() {
    const response = await fetch('https://jlp.deno.dev/', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            id: Math.random().toString(),
            jsonrpc: '2.0',
            method: 'jlp.furiganaservice.furigana',
            params: {
                q: q.value,
                grade: 1
            }
        })
    })

    html.value = generateRubyHtml(await response.json())
}

onMounted(() => {
})

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
        <v-textarea label="テキスト" v-model="q"></v-textarea>
        <v-btn @click="generate" color="primary" class="mr-4">生成</v-btn>
        <a href="https://developer.yahoo.co.jp/webapi/jlp/furigana/v2/furigana.html" target="_blank">Web Services by
            Yahoo! JAPAN</a>
        <div v-html="html" class="mt-4 output"></div>
    </v-container>
</template>