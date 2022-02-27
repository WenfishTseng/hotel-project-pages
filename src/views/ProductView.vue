<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-6">
        <img class="img-fluid" :src="product.imageUrl" :alt="product.title" />
      </div>
      <div class="col-sm-6">
        <span class="badge bg-primary rounded-pill">{{ product.category }}</span>
        <p>商品描述：{{ product.description }}</p>
        <p>商品內容：{{ product.content }}</p>
        <div class="h5" v-if="product.price === product.origin_price">
          {{ product.price }} 元
        </div>
        <div v-else>
          <div class="h6 text-decoration-line-through">
            原價 {{ product.origin_price }} 元
          </div>
          <div class="h5 text-danger fw-bold">
            現在只要 {{ product.price }} 元
          </div>
        </div>
        <div>
          <div class="input-group">
            <input
              type="number"
              class="form-control"
              v-model.number="qty"
              min="1"
            />
            <button type="button" class="btn btn-danger" @click="addCartQty">
              加入購物車
            </button>
          </div>
        </div>
      </div>
      <!-- col-sm-6 end -->
    </div>
  </div>
</template>

<script>
import emitter from '@/libs/emitter.js'

export default {
  data () {
    return {
      product: {},
      qty: 1
    }
  },
  methods: {
    getProduct () {
      const { id } = this.$route.params
      this.$http
        .get(
          `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/product/${id}`
        )
        .then((response) => {
          this.product = response.data.product
        })
        .catch((error) => {
          alert(error.response.data.message)
        })
    },
    addCartQty () {
      this.$http
        .post(
          `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/cart`,
          { data: { product_id: this.product.id, qty: this.qty } }
        )
        .then((response) => {
          this.cartData = response.data.data
          emitter.emit('get-cart')
          alert(response.data.message)
        })
        .catch((error) => {
          alert(error.response.data.message)
        })
    }
  },
  created () {
    this.getProduct()
  }
}
</script>
