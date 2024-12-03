<script setup>
import { ref } from 'vue'

const csr = ref(`-----BEGIN CERTIFICATE REQUEST-----
MIIDKjCCAhICAQAwgYQxFDASBgNVBAMMC2V4YW1wbGUuY29tMRQwEgYDVQQHDAtM
b3MgQW5nZWxlczETMBEGA1UECAwKQ2FsaWZvcm5pYTEUMBIGA1UECgwLRXhhbXBs
ZSBJbmMxCzAJBgNVBAsMAk5BMREwDwYJKoZIhvcNAQkBFgJOQTELMAkGA1UEBhMC
VVMwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDwkcOjk1kyo7cJ6oJi
Eh8ziUh+35NdgUXrJWhpb0sWddFYaq+VlwXp1fE9luQRc151zr4lLdGyV26LYGfT
A85S40q9IcSXklB+gQG5O+wdRisI76HMnQ/SFoHKOjsuaH+vosZvWureifuiTqly
kGsw6N+4i+O8RB/vQl9y6y1oLjeUOXxiBQkU97e9GBzXzvwSyXkrujArchRhkpd0
K7US0lkNbfV3As1UQxxkGDWXRHNYKJgjmTOQ0clzwBTL54gcgSGtwxEoVHgzgOGi
13+YTDvxDFKbZGEMnIAe0vHefiRIjXGujF2sbA+tJHw9lmNTleWI3XCwguosYWuY
R8eFAgMBAAGgYDBeBgkqhkiG9w0BCQ4xUTBPMAkGA1UdEwQCMAAwCwYDVR0PBAQD
AgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAWBgNVHREEDzANggtl
eGFtcGxlLmNvbTANBgkqhkiG9w0BAQsFAAOCAQEAvGmMT1+5rJ7F5cVDKs551BnH
dXOCTrhJ9gQ/IlYsT9h5IowJzkkaYwRDDy3vuxYCg3GrexhFEHrwQq62X7U46Wrg
U5NIKbFoOIEW18mhhhgeHOBqyNlMmCwZgDD99+O6NRtHAr/hMW5xDjHdcmtJKh0w
aqYyiEpR8YUAqod4pDT20IGdlDksoGP5bqfeIjm3Xqqz5SFj2zZMMA0RdNOibxMU
64cYx+iJ4+tfDg3mHfoR2YIvPz/WhrT0/9iKfh2nGH0xZWge5A28zMaGmEVSrt2a
DSBRNlhZ9XQn3d1W3Om0DiTv9448RuCZsOJSj8iWaDvhCKzei+WaLznFiCKyNA==
-----END CERTIFICATE REQUEST-----`)
const filename = ref('')
const content = ref('')
const downloaded = ref(false)

async function handleInput() {
    const csrBytes = forge.pki.pemToDer(csr.value).getBytes()
    const md5 = forge.md.md5.create().update(csrBytes).digest().toHex()
    const sha256 = forge.md.sha256.create().update(csrBytes).digest().toHex()
    filename.value = md5.toUpperCase() + '.txt'
    content.value = `${sha256}\nsectigo.com`
}

function downloadTextFile(text, filename) {
    const blob = new Blob([text], { type: 'text/plain' })
    const link = document.createElement('a')
    link.download = filename
    link.href = URL.createObjectURL(blob)
    link.click()
}

function download() {
    downloadTextFile(content.value, filename.value)
    downloaded.value = true
}

onMounted(() => {
    if (window.forge) requestIdleCallback(handleInput)
})

useHead({
    title: 'Sectigo CSR HASH Token Generator',
    script: [
        {
            src: 'https://cdn.jsdelivr.net/npm/node-forge/dist/forge.min.js',
            async: true,
            defer: true,
            onload: handleInput
        }
    ]
})
</script>

<template>
    <v-container>
        <v-textarea label="CSR" v-model="csr" rows="8" @input="handleInput"></v-textarea>
        <v-text-field label="filename" v-model="filename"></v-text-field>
        <v-textarea label="content" v-model="content" rows="3"></v-textarea>
        <a :href="'http://www.example.com/.well-known/pki-validation/' + filename">{{
            'http://www.example.com/.well-known/pki-validation/'
            + filename }}</a>
        <div class="d-flex align-center">
            <v-btn @click="download" color="secondary" class="mr-4">Download</v-btn>
            <a href="https://www.ssl247.com/knowledge-base/detail/domain-control-validation-dcv-methods/ka03l000000kfdIAAQ/"
                target="_blank">Sectigo
                documents</a>
        </div>
        <v-snackbar v-model="downloaded">Downloaded</v-snackbar>
    </v-container>
</template>
