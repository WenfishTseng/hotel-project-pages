<template>
  <div class="container">
    <div class="row row-cols-1 row-cols-lg-4 g-3">
      <div class="col" v-for="product in products" :key="product.id">
        <div class="card h-100">
          <img
            :src="product.imageUrl"
            class="card-img-top"
            :alt="product.title"
          />
          <div class="card-body">
            <h5 class="card-title">{{ product.title }}</h5>
            <p class="card-text">
              {{ product.content }}
            </p>
            <div class="h5" v-if="product.price === product.origin_price">
              {{ product.price }} 元
            </div>
            <div v-else>
              <del class="h6">原價 {{ product.origin_price }} 元</del>
              <div class="h5">現在只要 {{ product.price }} 元</div>
            </div>
          </div>
          <div class="card-footer bg-transparent border-top-0">
            <div class="btn-group btn-group-sm">
              <!-- 打開單一產品視窗 -->
              <router-link :to="`/products/${product.id}`"
                class="btn btn-outline-secondary"
              >
                <!-- 讀取圖示 若id相同 出現icon-->
                <i
                  class="fas fa-spinner fa-pulse"
                  v-show="isLoadingItem === product.id"
                ></i>
                查看更多
              </router-link>
              <button
                type="button"
                class="btn btn-danger"
                @click="addToCart(product.id)"
              >
                <i
                  class="fas fa-spinner fa-pulse"
                  v-show="isCartLoadingItem === product.id"
                ></i>
                加到購物車
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import emitter from '@/libs/emitter.js'

export default {
  name: 'ProductsView',
  data () {
    return {
      products: {},
      isLoadingItem: '',
      isCartLoadingItem: ''
    }
  },
  methods: {
    getProducts () {
      this.$http
        .get(
          `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/products/all`
        )
        .then((response) => {
          this.products = response.data.products
        })
        .catch((error) => {
          alert(error.data.message)
        })
    },
    addToCart (id, qty = 1) {
      this.isCartLoadingItem = id
      this.$http
        .post(
          `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/cart`,
          { data: { product_id: id, qty: qty } }
        )
        .then((response) => {
          this.cartData = response.data.data
          emitter.emit('get-cart')
          // 讀完清空id
          this.isCartLoadingItem = ''
          alert(response.data.message)
        })
        .catch((error) => {
          alert(error)
          console.dir(error)
        })
    },
    openProductPage (id) {
      // this.$router.push(`/${id}`)
    }
  },
  created () {
    this.getProducts()
  }
}
</script>

<style>
.card-img-top {
  object-fit: cover;
  height: 240px;
}
</style>
