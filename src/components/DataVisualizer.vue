<script setup>
import axios from "axios";
import { defineProps, ref } from "vue";

const props = defineProps({
  barcode: String,
});

const apiUrl = "https://world.openfoodfacts.net/api/v2/product";
const apiFields = "?fields=product_name,nutrition_grades,nutriscore_data";

const product = ref(null);
const errorMessage = ref(null);

const loadProduct = async () => {
  product.value = null;
  errorMessage.value = null;
  try {
    if (!props.barcode) {
      throw new Error("barcode is null");
    }

    const response = await axios.get(`${apiUrl}/${props.barcode}${apiFields}`);
    console.log(response.data);

    if (!response.data.product) {
      throw new Error(`Product data is null`);
    }

    const nutriscoreData = response.data.product.nutriscore_data;
    const extractedData = {};

    if (nutriscoreData.sugars) extractedData.sugar = nutriscoreData.sugars;
    if (nutriscoreData.sodium) extractedData.salt = nutriscoreData.sodium;
    if (nutriscoreData.proteins_value)
      extractedData.proteins = nutriscoreData.proteins_value;
    if (nutriscoreData.saturated_fat)
      extractedData.saturated_fat = nutriscoreData.saturated_fat;
    if (nutriscoreData.energy) extractedData.energy = nutriscoreData.energy;

    product.value = {
      product_name: response.data.product.product_name,
      nutrition_grades: response.data.product.nutrition_grades,
      nutrition_data: extractedData,
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
    <p v-if="product">
      Nutrition grade: {{ product.nutrition_grades.toUpperCase() }}
    </p>
    <h3 v-if="product">Nutrition data:</h3>
    <div v-if="product" class="item">
      <span class="label">Sugar: </span>
      <span class="value"> {{ product.nutrition_data.sugar }} g </span>
    </div>
    <div v-if="product" class="item">
      <span class="label">Salt: </span>
      <span class="value"> {{ product.nutrition_data.salt }} g </span>
    </div>
    <div v-if="product" class="item">
      <span class="label">Proteins: </span>
      <span class="value"> {{ product.nutrition_data.proteins }} g </span>
    </div>
    <div v-if="product" class="item">
      <span class="label">Saturated fat: </span>
      <span class="value"> {{ product.nutrition_data.saturated_fat }} g </span>
    </div>
    <div v-if="product" class="item">
      <span class="label">Energy: </span>
      <span class="value"> {{ product.nutrition_data.energy }} kJ </span>
    </div>
    <p v-if="errorMessage">Error: {{ errorMessage }}</p>
  </div>
</template>
