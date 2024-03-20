<script setup>
import axios from 'axios';
import { defineProps,ref } from 'vue';

const props = defineProps({
    barcode: String
})

const apiUrl = 'https://world.openfoodfacts.net/api/v2/product';
const apiFields = '?fields=product_name,nutrition_grades,nutriscore_data';

const product = ref(null);
const errorMessage = ref(null);

const loadProduct = async () => {
  try {
    if (!props.barcode) {
      throw new Error('barcode is null');
    }

    const response = await axios.get(`${apiUrl}/${props.barcode}${apiFields}`);

    if (!response.data.product) {
      throw new Error(`Product data is null`);
    }

    product.value = response.data.product;
  } catch (error) {
    errorMessage.value = `Error fetching product data: ${error.message}`;
  }
};

</script>

<template>
    <div id="product-info">
        <p v-if="product">Product name: {{ product.product_name }}</p>
        <p v-if="product">Nutrition grade: {{ product.nutrition_grades }}</p>
        <h3 v-if="product">Nutrition data: </h3>
        <div v-if="product" v-for="(value, key) in product.nutriscore_data" :key="key" class="item">
            <span class="label">{{ key }}:</span>
            <span class="value">{{ value }}</span>
        </div>
        <p v-if="errorMessage">Error: {{ errorMessage }}</p>
        <button @click="loadProduct">Load product</button>
    </div>
</template>

<script>

export default {
    data(){
        return{
            product:null,
            errorMessage:null,
        }
    },

    methods: {
//         loadProduct: async() => {
//             try {
//                 if (!barcode) {
//                     throw new Error('barcode is null')
//                 }

//                 const response = await axios.get(`${apiUrl}/${barcode}${apiFields}`)
                
//                 if (!response.data.product) {
//                     throw new Error('Product is null')
//                 }

//                 this.product.value = response.data.product

//             }catch (error) {
//                 console.log("error", error.message)
//                 this.errorMessage.value = `Error fetching product data: ${error.message}`;
//             }
        },
//     }
}


</script>