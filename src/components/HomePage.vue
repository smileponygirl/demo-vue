<template>
  <v-container class="fill-height">
    <v-responsive class="align-centerfill-height mx-auto">
      <div class="mb-5">
        <v-img
          :width="300"
          aspect-ratio="16/9"
          cover
          src="https://au.popmart.com/cdn/shop/files/fnl_62114af7-ac2d-4d3e-813e-469c39e377eb_280x.jpg?v=1704357922"
        ></v-img>
      </div>

      <v-dialog v-model="dialog" width="auto">
        <v-card
          min-width="500"
          prepend-icon="mdi-information-slab-circle-outline"
          text="Are you sure you want to buy this item?"
          title="Confirm"
        >
          <template v-slot:actions>
            <v-spacer></v-spacer>
            <v-btn @click="dialog = false"> Cancel </v-btn>
            <v-btn @click="buy()"> Continue, Buy it! </v-btn>
          </template>
        </v-card>
      </v-dialog>

      <div class="text-center">
        <v-snackbar v-model="snackbar" timeout="1500" color="success">
          Buy Success
          <template v-slot:actions>
            <v-btn variant="text" @click="snackbar = false"> Close </v-btn>
          </template>
        </v-snackbar>
      </div>

      <v-container v-for="product in productList" :key="product.id">
        {{ product.productName }}
        <v-row>
          <v-col
            cols="12"
            md="4"
            v-for="variant in product.attributes.variant"
            :key="variant.id"
          >
            <v-card class="mx-auto" rounded="lg">
              <v-img
                :src="variant.variantImage.data.attributes.url"
                cover
                min-height="300"
              ></v-img>
              <v-card-title>{{ variant.variantName }}</v-card-title>
              <v-card-subtitle>
                {{ variant.variantSKU }}
              </v-card-subtitle>
              <v-card-text
                style="
                  display: flex;
                  justify-content: space-between;
                  align-items: start;
                "
              >
                <v-chip :class="chipClass">{{ variant.stockQuantity }} </v-chip>
              </v-card-text>
              <v-card-actions>
                <v-btn
                  color="pink-lighten-1"
                  rounded="lg"
                  size="x-large"
                  block
                  border
                  class="mb-2"
                  @click="openConfirm(variant)"
                  >Buy Now!</v-btn
                >
              </v-card-actions>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-responsive>
  </v-container>
</template>

<script>
import axios from "axios";
export default {
  name: "productList",
  data() {
    return {
      snackbar: false,
      dialog: false,
      productList: [],
      itemSelected: null,
    };
  },
  created() {
    console.log("fetchItems");
    this.fetchItems();
  },
  methods: {
    async fetchItems() {
      try {
        const response = await axios.get("http://localhost:3500/v1/products");
        this.productList = response.data.data;
        console.log("response", response.data.data);
        console.log("productList", this.productList);
      } catch (error) {
        console.error("Error fetching items:", error);
      }
    },
    openConfirm(variant) {
      this.dialog = true;
      this.itemSelected = variant;
    },
    async buy() {
      // UPDATE
      console.log("BUY itemSelected", this.itemSelected);
      try {
        const newItem = JSON.parse(JSON.stringify(this.itemSelected));
        newItem.stockQuantity--;
        console.log("newItem", newItem);
        const response = await axios.put(
          `http://localhost:3500/v1/products/${newItem.id}`,
          newItem
        );
        this.dialog = false;
        this.snackbar = true;
        this.productList = response.data.data;
        console.log("response", response.data.data);
        console.log("productList", this.productList);
      } catch (error) {
        console.error("Error fetching items:", error);
      }
    },
  },
};
</script>
<style>
.chip-blue {
  background-color: blue;
  color: white;
}

.chip-red {
  background-color: red;
  color: white;
}

.chip-fade-enter-active,
.chip-fade-leave-active {
  transition: background-color 0.5s ease;
}

.chip-fade-enter,
.chip-fade-leave-to {
  background-color: transparent;
}
</style>
