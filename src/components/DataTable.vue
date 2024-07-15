<template>
  <v-container>
    <v-table hover>
      <thead>
        <tr>
          <th class="text-left"></th>
          <th
            v-for="storeName in data?.stores"
            :key="storeName"
            @click="chanageSorting(storeName)"
          >
            {{ storeName }}
            <SortIcon
              :selectedKey="selectedSorting.key"
              :selectedOrder="selectedSorting.order"
              :sortingKey="storeName"
            />
          </th>
        </tr>
        <tr>
          <th style="padding-top: 20px">
            <v-select
              label="Category"
              :items="data?.categories"
              variant="outlined"
              v-model="selectedCategory"
            ></v-select>
          </th>
          <th v-for="storeName in data?.stores">
            {{
              Object.keys(data?.items[selectedCategory]).reduce(
                (sum, productName) => {
                  return (sum +=
                    data?.items[selectedCategory][productName][storeName]);
                },
                0
              )
            }}
          </th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="productName in getSortedProductsInCategory(selectedCategory)"
          :key="productName"
        >
          <td style="font-weight: bold">{{ productName }}</td>
          <td v-for="storeName in data?.stores">
            {{ data?.items[selectedCategory][productName][storeName] }}
          </td>
        </tr>
      </tbody>
    </v-table>
  </v-container>
</template>
<script setup>
import { ref, toRaw } from "vue";

let selectedCategory = ref("Xbox");
let selectedSorting = { key: "PE", order: "ASC" };

const data = ref({
  items: [],
  stores: [],
  category: null,
});

const WestechAPI = {
  async fetch() {
    try {
      const response = await fetch("https://hiring.wdev.sk/fe-data");
      if (!response.ok) {
        throw new Error(`Response status: ${response.status}`);
      }

      const json = await response.json();

      // using sets so there is no duplicate value
      const stores = new Set();
      const categories = new Set();
      const data = {};

      json.forEach((item) => {
        stores.add(item.store);
        categories.add(item.category);
        if (!data[item.category]) {
          data[item.category] = {};
        }
        if (!data[item.category][item.product]) {
          data[item.category][item.product] = {};
        }
        data[item.category][item.product][item.store] = item.pcs;
      });

      return {
        items: data,
        stores: Array.from(stores),
        categories: Array.from(categories),
      };
    } catch (error) {
      console.error(error.message);
    }
  },
};

function chanageSorting(value) {
  if (
    !selectedSorting ||
    !selectedSorting.key ||
    selectedSorting.key != value
  ) {
    selectedSorting = { key: value, order: "DESC" };
  } else if (selectedSorting.key == value && selectedSorting.order == "DESC") {
    selectedSorting = { key: value, order: "ASC" };
  } else if (selectedSorting.key == value && selectedSorting.order == "ASC") {
    selectedSorting = {};
  }
  // set selectedCategory value emptz and back to trigger re-render of the table
  let oldSelectedCategory = selectedCategory.value;
  selectedCategory.value = "";
  selectedCategory.value = oldSelectedCategory;
}

const sortingProductsBySelectedSorting = (a, b) => {
  if (!selectedSorting.key) {
    return 0;
  } else if (selectedSorting.order == "ASC") {
    return b[selectedSorting.key] - a[selectedSorting.key];
  } else {
    return a[selectedSorting.key] - b[selectedSorting.key];
  }
};

function getSortedProductsInCategory(selectedCategory) {
  if (!data?._value.items[selectedCategory]) {
    return [];
  }

  // extracts part of the maped data that is displazed in table so it can sort them by
  let products = toRaw(data._value.items[selectedCategory]);
  let ArrayOfProducts = Object.keys(products).map((productName) => {
    let product = { name: productName };
    !data?._value.stores.forEach((storeName) => {
      product[storeName] = products[productName][storeName];
    });
    return product;
  });

  // using seleted sort key and order to reorder rows with products
  ArrayOfProducts.sort(sortingProductsBySelectedSorting);
  return ArrayOfProducts.map((product) => product.name);
}

WestechAPI.fetch().then((result) => (data.value = result));
</script>
