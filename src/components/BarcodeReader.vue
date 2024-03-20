<template>
    <div>
      <input type="file" accept="image/*" @change="handleFileInput">
      <div v-if="barcodeData">
        <p>Barcode Data: {{ barcodeData }}</p>
      </div>
    </div>
  </template>
  
  <script>
  import Quagga from 'quagga';
  
  export default {
    data() {
      return {
        barcodeData: null
      };
    },
    methods: {
      handleFileInput(event) {
        const file = event.target.files[0];
        if (file) {
          this.decodeBarcode(file);
        }
      },
      decodeBarcode(imageFile) {
        Quagga.decodeSingle(
          {
            src: URL.createObjectURL(imageFile),
            locate: true,
            inputStream: {
              size: 800
            },
            decoder: {
              readers: ["ean_reader"] // specify the barcode type you want to scan
            }
          },
          (result) => {
            if (result && result.codeResult) {
              this.barcodeData = result.codeResult.code;
            } else {
              this.barcodeData = "No barcode detected";
            }
          }
        );
      }
    }
  };
  </script>
  
  <style>
  /* Add any additional styling here */
  </style>
  