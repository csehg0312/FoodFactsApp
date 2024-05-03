<script setup>
import axios from 'axios';
import { defineProps, ref } from 'vue';

const props = defineProps({
  barcode: String
})

const apiUrl = 'https://world.openfoodfacts.net/api/v2/product';
const apiFields = '?fields=product_name,nutrition_grades,nutriscore_data';

const product = ref(null);
const errorMessage = ref(null);

const loadProduct = async () => {
  product.value = null; 
  errorMessage.value = null;
  try {
    if (!props.barcode) {
      throw new Error('barcode is null');
    }

    const response = await axios.get(`${apiUrl}/${props.barcode}${apiFields}`);

    if (!response.data.product) {
      throw new Error(`Product data is null`);
    }

    const nutriscoreData = response.data.product.nutriscore_data;
    const extractedData = {
      sugar: nutriscoreData.sugars,
      salt: nutriscoreData.sodium,
      proteins: nutriscoreData.proteins
    };

    product.value = {
      product_name: response.data.product.product_name,
      nutrition_grades: response.data.product.nutrition_grades,
      nutrition_data: extractedData
    };
  } catch (error) {
    errorMessage.value = `Error fetching product data: ${error.message}`;
  }
};
</script>

<template>
  <div id="loadfunction">
    <button @click="loadProduct">Load product</button>
  </div>
  <div id="product-info">
    <p v-if="product">Product name: {{ product.product_name }}</p>
    <p v-if="product">Nutrition grade: {{ product.nutrition_grades }}</p>
    <h3 v-if="product">Nutrition data: </h3>
    <div v-if="product" class="item">
      <span class="label">Sugar: </span>
      <span class="value"> {{ product.nutrition_data.sugar }}</span>
    </div>
    <div v-if="product" class="item">
      <span class="label">Salt: </span>
      <span class="value"> {{ product.nutrition_data.salt }}</span>
    </div>
    <div v-if="product" class="item">
      <span class="label">Proteins: </span>
      <span class="value"> {{ product.nutrition_data.proteins }}</span>
    </div>
    <p v-if="errorMessage">Error: {{ errorMessage }}</p>
  </div>
</template>