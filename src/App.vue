<template>
  <div class="scanner-container">
    <h2>Сканер QR-коду</h2>
    
    <div id="reader"></div>

    <div v-if="scanResult" class="result">
      <h3>Результат сканування:</h3>
      <p><strong>{{ scanResult }}</strong></p>
      <button @click="scanResult = null">Сканувати знову</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { Html5QrcodeScanner } from "html5-qrcode"

const scanResult = ref<string | null>(null)
let scanner: Html5QrcodeScanner | null = null

const onScanSuccess = (decodedText: string) => {
  // Зупиняємо сканер після успішного зчитування (опціонально)
  scanResult.value = decodedText

  // Додамо вібрацію для кращого UX
  if (navigator.vibrate) {
    navigator.vibrate(100)
  }
}

const onScanFailure = (error: string) => {
  // Тут можна логувати помилки, але зазвичай вони надто часті (поки камера шукає код)
  // console.warn(`QR error: ${error}`);
}

onMounted(() => {
  scanner = new Html5QrcodeScanner(
    "reader", // ID елемента в шаблоні
    { 
      fps: 10, 
      qrbox: { width: 250, height: 250 },
      rememberLastUsedCamera: true
    },
    /* verbose= */ false
  )
  scanner.render(onScanSuccess, onScanFailure)
})

onUnmounted(() => {
  if (scanner) {
    // Важливо: звільняємо камеру при виході з компонента
    scanner.clear().catch(error => {
      console.error("Failed to clear html5QrcodeScanner", error)
    })
  }
})
</script>

<style scoped>
.scanner-container {
  max-width: 500px;
  margin: 0 auto;
  text-align: center;
}
#reader {
  width: 100%;
  border-radius: 8px;
  overflow: hidden;
}
.result {
  margin-top: 20px;
  padding: 15px;
  background-color: #f0fdf4;
  border: 1px solid #bbf7d0;
  border-radius: 8px;
  word-break: break-all;
}
button {
  margin-top: 10px;
  padding: 8px 16px;
  cursor: pointer;
}
</style>