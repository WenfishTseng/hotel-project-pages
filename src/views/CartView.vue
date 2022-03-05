<template>
  <Loading
    v-model:active="isLoading"
    :can-cancel="true"
    :on-cancel="onCancel"
    :is-full-page="fullPage"
  ></Loading>
  <div class="container mt-3">
    <div class="text-end">
      <button
        class="btn btn-outline-dark"
        type="button"
        @click="removeAllCarts"
        :disabled="cartData.total === 0"
      >
        清空購物車
      </button>
    </div>
    <table class="table align-middle">
      <thead>
        <tr>
          <th></th>
          <th>品名</th>
          <th style="width: 150px">數量/單位</th>
          <th class="text-end">單價</th>
        </tr>
      </thead>
      <tbody>
        <!-- 確認購物車資料有沒有存在 -->
        <template v-if="cartData.carts">
          <tr v-for="item in cartData.carts" :key="item.id">
            <td>
              <button
                type="button"
                @click="removeCartItem(item.id)"
                class="btn btn-outline-danger btn-sm"
              >
                <i
                  class="fas fa-spinner fa-pulse"
                  v-show="item.id === isDeleteLoadingItem"
                ></i>
                x
              </button>
            </td>
            <td>
              {{ item.product.title }}
              <div class="text-success">已套用優惠券</div>
            </td>
            <!-- 盡量不要讓使用者填東西 -->
            <td>
              <div class="input-group input-group-sm">
                <div class="input-group mb-3">
                  <select
                    name="cartNum"
                    id="cart"
                    class="form-selest"
                    v-model="item.qty"
                    @change="updateCart(item)"
                    :disabled="isCartLoadingItem === item.id"
                  >
                    <option v-for="i in 20" :value="i" :key="i + 'cart'">
                      {{ i }}
                    </option>
                  </select>
                  <span class="input-group-text" id="basic-addon2">{{
                    item.product.unit
                  }}</span>
                </div>
              </div>
            </td>
            <td class="text-end">
              <small> {{ item.product.origin_price }} </small>
              <span class="ms-3 text-success"
                >折扣價： {{ item.product.price }}</span
              >
            </td>
          </tr>
        </template>
      </tbody>
      <tfoot>
        <tr>
          <td colspan="3" class="text-end">總計</td>
          <td class="text-end">{{ cartData.total }}</td>
        </tr>
      </tfoot>
    </table>
    <div class="text-end">
      <button
        class="btn btn-danger"
        type="button"
        @click="goOrderPage"
        :disabled="cartData.total === 0"
      >
        確認訂單，結帳
      </button>
    </div>

  </div>
</template>

<script>
import emitter from '@/libs/emitter.js'
import Loading from 'vue-loading-overlay'
import 'vue-loading-overlay/dist/vue-loading.css'

export default {
  data () {
    return {
      // 購物車列表
      cartData: {},
      isLoading: false
    }
  },
  components: { Loading },
  methods: {
    getCarts () {
      this.isLoading = true
      this.$http
        .get(`${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/cart`)
        .then((response) => {
          this.cartData = response.data.data
          emitter.emit('get-cart')
          this.isLoading = false
        })
        .catch((error) => {
          this.isLoading = false
          alert(error)
        })
    },
    updateCart (item) {
      this.isCartLoadingItem = item.id
      this.$http
        .put(
          `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/cart/${item.id}`,
          { data: { product_id: item.id, qty: item.qty } }
        )
        .then((response) => {
          this.getCarts()
          this.isCartLoadingItem = ''
        })
        .catch((error) => {
          alert(error.data)
        })
    },
    removeCartItem (id) {
      this.isDeleteLoadingItem = id
      this.$http
        .delete(
          `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/cart/${id}`
        )
        .then((response) => {
          this.getCarts()
          // 讀完清空id
          this.isDeleteLoadingItem = ''
        })
        .catch((error) => {
          alert(error.data.message)
        })
    },
    removeAllCarts () {
      this.$http
        .delete(
          `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/carts`
        )
        .then((response) => {
          this.getCarts()
        })
        .catch((error) => {
          alert(error.data.message)
        })
    },
    goOrderPage () {
      this.$router.push('/order')
    }
  },
  created () {
    this.getCarts()
  }
}
</script>
