<script setup>
import { ref } from 'vue'

const video = ref(null)
const canvas = ref(null)
const loaded = ref(false)

async function handleChange(e) {
    if (!e.target.files.length) return
    video.value.src = URL.createObjectURL(e.target.files[0])
}

function handleLoadeddata() {
    loaded.value = true
}

function pick() {
    canvas.value.width = video.value.videoWidth
    canvas.value.height = video.value.videoHeight
    canvas.value.getContext('2d').drawImage(video.value, 0, 0, video.value.videoWidth, video.value.videoHeight)
    const link = document.createElement('a')
    link.download = 'image.png'
    link.href = canvas.value.toDataURL()
    link.click()
}

function previousFrame() {
    video.value.currentTime -= 1 / 30
}

function nextFrame() {
    video.value.currentTime += 1 / 30
}

useHead({
    title: 'Video Frame Picker',
})
</script>

<template>
    <v-container>
        <v-file-input label="Choose video file or drop here" @change="handleChange"></v-file-input>
        <video ref="video" style="max-width: 720px" class="w-100" controls @loadeddata="handleLoadeddata"
            v-show="loaded"></video>
        <div>
            <v-btn @click="previousFrame" v-show="loaded" color="secondary" class="ma-2">
                < Previous frame</v-btn>
                    <v-btn @click="nextFrame" v-show="loaded" color="secondary" class="ma-2">Next frame ></v-btn>
                    <v-btn @click="pick" v-show="loaded" color="primary" class="ma-2">Pick frame</v-btn>
                    <canvas ref="canvas" hidden></canvas>
        </div>
    </v-container>
</template>
