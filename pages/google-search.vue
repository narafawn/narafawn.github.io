<script setup>
import { ref } from 'vue'

const q = ref('Donald Trump')
const key = ref('AIzaSyBm0aE3RmW2R7no-pRLcWbEON7XKnurphk')
const lr = ref({ title: '日本語', value: 'lang_ja' })
const lrItems = ref([
    { title: 'アラビア語', value: 'lang_ar' },
    { title: 'ブルガリア語', value: 'lang_bg' },
    { title: 'カタロニア語', value: 'lang_ca' },
    { title: 'チェコ語', value: 'lang_cs' },
    { title: 'デンマーク語', value: 'lang_da' },
    { title: 'ドイツ語', value: 'lang_de' },
    { title: 'ギリシャ語', value: 'lang_el' },
    { title: '英語', value: 'lang_en' },
    { title: 'スペイン語', value: 'lang_es' },
    { title: 'エストニア語', value: 'lang_et' },
    { title: 'フィンランド語', value: 'lang_fi' },
    { title: 'フランス語', value: 'lang_fr' },
    { title: 'クロアチア語', value: 'lang_hr' },
    { title: 'ハンガリー語', value: 'lang_hu' },
    { title: 'インドネシア語', value: 'lang_id' },
    { title: 'アイスランド語', value: 'lang_is' },
    { title: 'イタリア語', value: 'lang_it' },
    { title: 'ヘブライ語', value: 'lang_iw' },
    { title: '日本語', value: 'lang_ja' },
    { title: '韓国語', value: 'lang_ko' },
    { title: 'リトアニア語', value: 'lang_lt' },
    { title: 'ラトビア語', value: 'lang_lv' },
    { title: 'オランダ語', value: 'lang_nl' },
    { title: 'ノルウェー語', value: 'lang_no' },
    { title: 'ポーランド語', value: 'lang_pl' },
    { title: 'ポルトガル語', value: 'lang_pt' },
    { title: 'ルーマニア語', value: 'lang_ro' },
    { title: 'ロシア語', value: 'lang_ru' },
    { title: 'スロバキア語', value: 'lang_sk' },
    { title: 'スロベニア語', value: 'lang_sl' },
    { title: 'セルビア語', value: 'lang_sr' },
    { title: 'スウェーデン語', value: 'lang_sv' },
    { title: 'トルコ語', value: 'lang_tr' },
    { title: '中国語（簡体）', value: 'lang_zh-CN' },
    { title: '中国語（繁体）', value: 'lang_zh-TW' },
])
const url = ref('https://customsearch.googleapis.com/customsearch/v1?key=AIzaSyBm0aE3RmW2R7no-pRLcWbEON7XKnurphk&q=Donald+Trump&cx=409f003bf50414412&lr=lang_ja')
const highlightedJson = ref('')

async function search() {
    console.log(lr.value.value)
    if (!q.value) return
    url.value = 'https://customsearch.googleapis.com/customsearch/v1?' + new URLSearchParams({
        key: key.value,
        q: q.value,
        cx: '409f003bf50414412',
        lr: lr.value.value
    })
    let json = await fetch(url.value).then(r => r.text())
    highlightedJson.value = Prism.highlight(json, Prism.languages.json, 'json')
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

const handleInput = debounce(search, 1000)

async function onload() {
    const json = await fetch('https://gist.githubusercontent.com/GitHub30/e70e2f47ceaae80331d5ef34d97278e1/raw/2f9d4fc03deb8394174a13ac2ee2970c8fd42b9a/customsearch.sample.json').then(r => r.text())
    highlightedJson.value = Prism.highlight(json, Prism.languages.json, 'json')
}

useHead({
    title: 'Google Search',
    link: [
        {
            rel: 'stylesheet',
            href: 'https://cdn.jsdelivr.net/npm/prismjs@1.x/themes/prism.min.css'
        }
    ],
    script: [
        {
            src: 'https://cdn.jsdelivr.net/npm/prismjs@1.x/prism.min.js',
            async: true,
            defer: true,
            'data-manual': true
        },
        {
            src: 'https://cdn.jsdelivr.net/npm/prismjs@1.x/components/prism-json.min.js',
            async: true,
            defer: true,
            'data-manual': true,
            onload
        },
    ]
})
</script>

<template>
    <v-container>
        <div class="d-flex">
            <v-text-field v-model="q" label="q" width="200px" class="mr-2" @input="handleInput"></v-text-field>
            <v-text-field v-model="key" label="key" width="200px" class="mr-2" @input="handleInput"></v-text-field>
            <v-combobox v-model="lr" label="lr" :items="lrItems" width="200px"
                @update:modelValue="handleInput"></v-combobox>
        </div>
        <a :href="url" target="_blank">{{ url }}</a>
        <br>
        <a href="https://developers.google.com/custom-search/v1/introduction#identify_your_application_to_google_with_api_key"
            target="_blank">Create key (Free)</a>
        <a class="ml-2" href="https://programmablesearchengine.google.com/controlpanel/create" target="_blank">Create cx
            (Free)</a>
        <a class="ml-2" href="https://developers.google.com/custom-search/v1/reference/rest/v1/cse/list"
            target="_blank">Documents</a>
        <pre><code class="language-json" v-html="highlightedJson"></code></pre>
    </v-container>
</template>
