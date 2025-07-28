<template>
  <v-container class="pa-4">
    <v-row justify="center" class="mb-4">
      <v-col cols="12" md="8" class="text-center">
        <v-btn
          @click="startScanner"
          color="primary"
          class="ma-2"
          :disabled="isScanning"
          size="large"
          elevation="2"
          rounded
        >
          Start Scanner
        </v-btn>

        <v-btn
          @click="stopScanner"
          color="error"
          class="ma-2"
          :disabled="!isScanning"
          size="large"
          elevation="2"
          rounded
        >
          Stop Scanner
        </v-btn>
      </v-col>
    </v-row>

    <v-row justify="center">
      <v-col cols="12" md="8">
        <div id="video-container" class="scanner-box elevation-3"></div>
      </v-col>
    </v-row>

    <v-row justify="center" class="mt-4">
      <v-col cols="12" md="8">
        <v-alert
          v-if="scannedData"
          type="success"
          variant="outlined"
          prominent
          border="start"
          color="green"
        >
          <strong>Scanned:</strong>
          <template v-if="isLink(scannedData)">
            <a
              :href="scannedData"
              target="_blank"
              rel="noopener noreferrer"
              style="color: green; text-decoration: underline; cursor: pointer"
            >
              {{ scannedData }}
            </a>
          </template>
          <template v-else>
            {{ scannedData }}
          </template>
        </v-alert>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup lang="ts">
import { ref, onBeforeUnmount, nextTick } from "vue";
import { Html5Qrcode, Html5QrcodeSupportedFormats } from "html5-qrcode";

const scannedData = ref<string | null>(null);
const isScanning = ref(false);
let scanner: Html5Qrcode | null = null;

const SCANNER_ID = "video-container";

const isLink = (text: string | null): boolean => {
  if (!text) return false;
  return /^https?:\/\/[\w\-._~:/?#[\]@!$&'()*+,;=%]+$/i.test(text);
};

const startScanner = async () => {
  if (scanner || isScanning.value) return;

  await nextTick();

  const container = document.getElementById(SCANNER_ID);
  if (!container) {
    console.error("Scanner container not found.");
    return;
  }

  try {
    scanner = new Html5Qrcode(SCANNER_ID);
    isScanning.value = true;

    await scanner.start(
      { facingMode: "environment" },
      {
        fps: 10,
        qrbox: { width: 250, height: 250 },
        formatsToSupport: [
          Html5QrcodeSupportedFormats.QR_CODE,
          Html5QrcodeSupportedFormats.CODE_128,
          Html5QrcodeSupportedFormats.EAN_13,
        ],
      },
      (decodedText: string) => {
        scannedData.value = decodedText;
        console.log("Scanned:", decodedText);
        stopScanner();
      },
      (errorMessage) => {
        console.debug("Scan error:", errorMessage);
      }
    );
  } catch (err) {
    console.error("Failed to start scanner:", err);
    isScanning.value = false;
  }
};

const stopScanner = async () => {
  if (scanner) {
    try {
      await scanner.stop();
      await scanner.clear();
    } catch (err) {
      console.error("Error stopping scanner:", err);
    } finally {
      scanner = null;
      isScanning.value = false;
    }
  }
};

onBeforeUnmount(() => {
  stopScanner();
});
</script>

<style scoped>
.scanner-box {
  width: 100%;
  height: 400px;
  border-radius: 12px;
  background-color: #000;
  overflow: hidden;
  position: relative;
  border: 1px solid #ccc;
}

#video-container video {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 12px;
}
</style>
