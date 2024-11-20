<script setup>
import { ref } from 'vue'

const csrPem = ref('')
const privateKeyPem = ref('')
const countryName = ref('GB')
const stateOrProvinceName = ref('Somerset')
const localityName = ref('Glastonbury')
const organizationName = ref('The Brain Room Limited')
const organizationalUnitName = ref('PHP Documentation Team')
const commonName = ref('example.com')
const csrCopied = ref(false)
const privateKeyCopied = ref(false)

function generate() {
    const keys = forge.pki.rsa.generateKeyPair(2048)
    const csr = forge.pki.createCertificationRequest()
    csr.publicKey = keys.publicKey
    csr.setSubject([
        { name: 'countryName', value: countryName.value },
        { name: 'stateOrProvinceName', value: stateOrProvinceName.value },
        { name: 'localityName', value: localityName.value },
        { name: 'organizationName', value: organizationName.value },
        { name: 'organizationalUnitName', value: organizationalUnitName.value },
        { name: 'commonName', value: commonName.value },
    ])
    csr.sign(keys.privateKey)
    csrPem.value = forge.pki.certificationRequestToPem(csr)
    privateKeyPem.value = forge.pki.privateKeyToPem(keys.privateKey)
}

function copy() {
    navigator.clipboard.writeText(csrPem.value)
    csrCopied.value = true
}

function privateKeyCopy() {
    navigator.clipboard.writeText(privateKeyPem.value)
    privateKeyCopied.value = true
}

useHead({
    title: 'CSR Generator',
    script: [
        {
            src: 'https://cdn.jsdelivr.net/npm/node-forge/dist/forge.min.js',
            async: true,
            defer: true
        }
    ]
})
</script>

<template>
    <v-container>
        <v-text-field v-model="countryName" label="Country Name"></v-text-field>
        <v-text-field v-model="stateOrProvinceName" label="State Or Province Name"></v-text-field>
        <v-text-field v-model="localityName" label="Locality Name"></v-text-field>
        <v-text-field v-model="organizationName" label="Organization Name"></v-text-field>
        <v-text-field v-model="organizationalUnitName" label="Organizational Unit Name"></v-text-field>
        <v-text-field v-model="commonName" label="Common Name"></v-text-field>
        <v-btn @click="generate" color="primary" class="ma-2">Generate</v-btn>
        <v-btn @click="copy" color="secondary" class="ma-2">CSR Copy</v-btn>
        <v-btn @click="privateKeyCopy" color="secondary" class="ma-2">Private Key Copy</v-btn>
        <v-textarea label="CSR" v-model="csrPem"></v-textarea>
        <v-textarea label="Private Key" v-model="privateKeyPem"></v-textarea>
        <v-snackbar v-model="csrCopied">CSR Copied!</v-snackbar>
        <v-snackbar v-model="privateKeyCopied">Private Key Copied!</v-snackbar>
    </v-container>
</template>