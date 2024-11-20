<template>
  <NuxtLayout>
    <v-app>
      <v-app-bar>
        <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>
        <v-app-bar-title>My Toolkit Online</v-app-bar-title>
        <v-btn href="https://github.com/GitHub30/mytoolkit.app" icon="mdi-github"></v-btn>
      </v-app-bar>

      <v-navigation-drawer v-model="drawer">
        <v-list>
          <v-list-item title="Password Generator" to="password-generator"></v-list-item>
          <v-list-item title="UUID Generator" to="uuid-generator"></v-list-item>
          <v-list-item title="file.gz Extractor" to="gz-extractor"></v-list-item>
          <v-list-item title="TOTP Generator" to="totp-generator"></v-list-item>
          <v-list-item title="CSR Generator" to="csr-generator"></v-list-item>
          <v-list-item title="CSR Decoder" to="csr-decoder"></v-list-item>
          <v-list-item title="SSH Key Generator" to="ssh-key-generator"></v-list-item>
          <v-list-item title="Count Lines" to="count-lines"></v-list-item>
          <v-list-item title="Replace text" to="replace-text"></v-list-item>
          <v-list-item title="Video Frame Picker" to="video-frame-picker"></v-list-item>
          <v-list-item title="unserialize.php" to="unserialize-php"></v-list-item>
          <v-list-item title="OCR" to="ocr"></v-list-item>
        </v-list>
      </v-navigation-drawer>

      <v-main>
        <NuxtPage />
      </v-main>
    </v-app>
  </NuxtLayout>
</template>

<script setup>
import { ref } from 'vue'

const drawer = ref(null)
</script>
