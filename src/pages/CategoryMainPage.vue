import { response } from 'express';
<template>
  <main class="content container">
    <div class="content__top">
      <ul class="breadcrumbs">
        <ul class="breadcrumbs">
          <li class="breadcrumbs__item">
            <router-link class="breadcrumbs__link" :to="{ name: 'main' }">
              Каталог
            </router-link>
          </li>
          <li class="breadcrumbs__item">
            <a class="breadcrumbs__link">
              {{ this.$route.params.category }}
            </a>
          </li>
        </ul>
      </ul>
    </div>

    <div class="content__top content__top--catalog">
      <h1 class="content__title">
        Каталог
      </h1>
      <span class="content__info">
        {{ countProducts }} товаров
      </span>
    </div>

    <div class="content__catalog">
      <ProductFilter v-model:price-from="filterPriceFrom" v-model:price-to="filterPriceTo"
        v-model:catedory-id="filterCategoryId" v-model:color-filter="filterColor" />

      <section class="catalog">
        <LoadingPreloader v-if="productsLoading" />
        <div class="error" v-if="productsLoadingFailed">
          <h1 class="error__header">Произошла ошибка при загрузке товара</h1>
          <button class="error__btn" @click.prevent="loadProducts">Попробуйте еще раз</button>
        </div>

        <ProductList :products="products" />

        <BasePagination v-model="page" :count='countProducts' :per-page="productsPerPage" />
      </section>

    </div>
  </main>
</template>

<script>
import ProductList from '@/components/ProductList.vue';
import BasePagination from '@/components/BasePagination.vue';
import ProductFilter from '@/components/ProductFilter.vue';
import axios from 'axios';
import { API_BASE_URL } from '../config';
import LoadingPreloader from '@/components/LoadingPreloader.vue';



export default {
  components: { ProductList, BasePagination, ProductFilter, LoadingPreloader },
  data() {
    return {
      filterPriceFrom: null,
      filterPriceTo: null,
      filterColor: null,
      page: 1,
      productsPerPage: 12,
      filterCategoryId: this.$route.params.id,
      productsData: null,

      productsLoading: false,

      productsLoadingFailed: false,
    }
  },
  computed: {
    products() {
      return this.productsData ?
        this.productsData.items.map(product => {
          return {
            ...product,
            image: product.preview.file.url
          }
            
        })
        : [];


    },

    countProducts() {

      return this.productsData ? this.productsData.pagination.total : 0;
    }
  },
  methods: {
    loadProducts() {
      this.productsData=null,
      this.productsLoading = true;
      this.productsLoadingFailed = false;
      clearTimeout(this.loadProductsTimer);
      this.loadProductsTimer = setTimeout(() => {
        axios.get(API_BASE_URL + "/api/products", {
          params: {
            categoryId: this.filterCategoryId,
            page: this.page,
            limit: this.productsPerPage,
            minPrice: this.filterPriceFrom,
            maxPrice: this.filterPriceTo,
            colorId: this.filterColor,
          }
        })
          .then(response => this.productsData = response.data)
          .catch(() => this.productsLoadingFailed = true)
          .then(() => this.productsLoading = false)


      }, 500);
    }
  },
  watch: {
    page() {
      this.loadProducts();
    },
    filterPriceFrom() {
      this.loadProducts();
    },
    filterPriceTo() {
      this.loadProducts();
    },
    filterCategoryId() {
      this.loadProducts();
    },
    filterColor() {
      this.loadProducts();
    },
  },
  created() {
    this.loadProducts();
  }
};
</script>

<style >
.error {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.error__header {
  color: indigo;
  font-size: 30px;
}

.error__btn {
  border-radius: 10px;
  padding: 30px;
  font-size: 21px;
  background-color: slateblue;
}
</style>