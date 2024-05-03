<script setup>
import axios from "axios";
import { defineProps, ref } from "vue";

const props = defineProps({
  barcode: String,
});

const apiUrl = "https://world.openfoodfacts.net/api/v2/product";
const apiFields =
  "?fields=product_name,nutrition_grades,nutriscore_data,nutrition,images";

const product = ref(null);
const errorMessage = ref(null);
const imageUrl = ref(null);

const loadProduct = async () => {
  product.value = null;
  errorMessage.value = null;
  imageUrl.value = null;
  try {
    if (!props.barcode) {
      throw new Error("barcode is null");
    }

    const response = await axios.get(`${apiUrl}/${props.barcode}${apiFields}`);
    // console.log(response.data);

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

    const getImageUrl = (productData, imageName, resolution = "full") => {
      if (
        !productData.product.images ||
        !productData.product.images[imageName]
      ) {
        return null;
      }
      const baseUrl = "https://images.openfoodfacts.org/images/products";
      let prodcode = productData.code;
      if (prodcode.length > 8) {
        prodcode = prodcode.replace(/(...)(...)(...)(.*)/, "$1/$2/$3/$4");
      }
      let filename;
      if (/^\d+$/.test(imageName)) {
        const resolutionSuffix = resolution === "full" ? "" : `.${resolution}`;
        filename = `${imageName}${resolutionSuffix}.jpg`;
      } else {
        const rev = productData.product.images[imageName].rev;
        filename = `${imageName}.${rev}.${resolution}.jpg`;
      }
      return `${baseUrl}/${prodcode}/${filename}`;
    };

    const productData = response.data;
    const imageName = 1;

    imageUrl.value = getImageUrl(productData, imageName, 400);
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
    <img v-if="imageUrl" :src="imageUrl" />
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
      <span class="label">Calories: </span>
      <span class="value"> {{ product.nutrition_data.energy }} kJ </span>
    </div>
    <p v-if="errorMessage">Error: {{ errorMessage }}</p>
  </div>
</template>
