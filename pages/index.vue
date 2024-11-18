<template>
    <v-container>
        <v-btn href="password-generator" class="mr-4" variant="outlined" color="primary">Password Generator</v-btn>
        <v-btn href="uuid-generator" class="mr-4" variant="outlined" color="primary">UUID Generator</v-btn>
        <v-btn href="gz-extractor" class="mr-4" variant="outlined" color="primary">file.gz Extractor</v-btn>
        <v-btn href="totp-generator" class="mr-4" variant="outlined" color="primary">TOTP Generator</v-btn>
        <v-btn href="video-frame-picker" class="mr-4" variant="outlined" color="primary">Video Frame Picker</v-btn>
    </v-container>
</template>

<script setup>
useHead({
    title: 'My App',
})
</script>
