<script setup>
import { ref } from 'vue'

function generateUUID() {
    return ([1e7] + -1e3 + -4e3 + -8e3 + -1e11).replace(/[018]/g, c =>
        (c ^ crypto.getRandomValues(new Uint8Array(1))[0] & 15 >> c / 4).toString(16)
    );
}

const uuid = ref('')
const history = ref([])
const copied = ref(false)
const headers = [
    { title: 'UUID', key: 'uuid' },
    { title: 'CreatedAt', key: 'createdAt', value: item => new Date(item.createdAt).toLocaleString('sv') },
]

function generate() {
    uuid.value = generateUUID()
    history.value = [{ uuid: uuid.value, createdAt: new Date() }, ...history.value]
    localStorage.setItem('uuid.history', JSON.stringify(history.value))
}

function copy() {
    const i = document.body.appendChild(document.createElement('input'))
    i.value = uuid.value
    i.select()
    document.execCommand('copy')
    document.body.removeChild(i)
    copied.value = true
}

onMounted(() => {
    history.value = JSON.parse(localStorage.getItem('uuid.history')) ?? []
    generate()
})

function handleClickRow(event, row) {
    const i = document.body.appendChild(document.createElement('input'))
    i.value = row.item.uuid
    i.select()
    document.execCommand('copy')
    document.body.removeChild(i)
    copied.value = true
}

useHead({
    title: 'UUID Generator',
})
</script>

<template>
    <v-container>
        <v-btn @click="generate" color="primary">Generate</v-btn>
        <div class="d-flex align-center mt-4">
            <v-text-field v-model="uuid" label="UUID" class="flex-grow-0" width="400" style="max-width: 500px;"
                hide-details></v-text-field>
            <v-btn @click="copy" color="secondary" class="ml-4">Copy</v-btn>
        </div>
        <v-data-table :headers="headers" :items="history" @click:row="handleClickRow"></v-data-table>
        <v-snackbar v-model="copied">Copied!</v-snackbar>
    </v-container>
</template>