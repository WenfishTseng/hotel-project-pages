<template>
  <div class="container mt-3">
    <div class="text-end">
      <button
        class="btn btn-outline-success"
        type="button"
        @click="goBackPage"
      >
        上一步
      </button>
    </div>
    <table class="table align-middle">
      <thead>
        <tr>
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
  </div>
  <div class="container mt-3">
    <Form v-slot="{ errors }" ref="form" @submit="onSubmit">
      <div class="mb-3">
        <label for="email" class="form-label">收件人信箱</label>
        <Field
          id="email"
          name="email"
          type="email"
          class="form-control"
          :class="{ 'is-invalid': errors['email'] }"
          placeholder="請輸入 Email"
          rules="email|required"
          v-model="form.user.email"
        ></Field>
        <error-message name="email" class="invalid-feedback"></error-message>
      </div>

      <div class="mb-3">
        <label for="name" class="form-label">收件人姓名</label>
        <Field
          id="name"
          name="姓名"
          type="text"
          class="form-control"
          :class="{ 'is-invalid': errors['姓名'] }"
          placeholder="請輸入姓名"
          rules="min:8||max:12||required"
          v-model="form.user.name"
        ></Field>
        <error-message name="姓名" class="invalid-feedback"></error-message>
      </div>

      <div class="mb-3">
        <label for="tel" class="form-label">收件人手機號碼</label>
        <Field
          id="tel"
          name="電話"
          type="tel"
          class="form-control"
          :class="{ 'is-invalid': errors['電話'] }"
          placeholder="請輸入電話"
          v-model="form.user.tel"
          :rules="isFormPhone"
        ></Field>
        <error-message name="電話" class="invalid-feedback"></error-message>
      </div>

      <div class="mb-3">
        <label for="address" class="form-label">收件人地址</label>
        <Field
          id="address"
          name="地址"
          type="text"
          class="form-control"
          :class="{ 'is-invalid': errors['地址'] }"
          placeholder="請輸入地址"
          rules="required"
          v-model="form.user.address"
        ></Field>
        <error-message name="地址" class="invalid-feedback"></error-message>
      </div>

      <div class="mb-3">
        <label for="message" class="form-label">留言</label>
        <textarea
          id="message"
          class="form-control"
          cols="30"
          rows="10"
          v-model="form.message"
        ></textarea>
      </div>

      <div class="text-end">
        <!-- 若有錯誤或購物車數量為0不可以送出表單 -->
        <button
          type="submit"
          class="btn btn-danger"
          :disabled="Object.keys(errors).length > 0 || cartData.total === 0"
        >
          送出訂單
        </button>
      </div>
    </Form>
  </div>
</template>

<script>
import emitter from '@/libs/emitter.js'

export default {
  data () {
    return {
      // 購物車列表
      cartData: {},
      form: {
        user: {
          name: '',
          email: '',
          tel: '',
          address: ''
        },
        message: ''
      }
    }
  },
  methods: {
    getCarts () {
      this.$http
        .get(`${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/cart`)
        .then((response) => {
          this.cartData = response.data.data
          emitter.emit('get-cart')
        })
        .catch((error) => {
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
    isFormPhone (value) {
      const phoneNumber = /^(09)[0-9]{8}$/
      return phoneNumber.test(value) ? true : '需要正確的電話號碼'
    },
    onSubmit () {
      const order = this.form
      this.$http
        .post(
          `${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/order`,
          { data: order }
        )
        .then((response) => {
          alert(response.data.message)
          this.$refs.form.resetForm()
          this.getCarts()
          this.$router.push('/products')
        })
        .catch((error) => {
          console.dir(error)
        })
    },
    goBackPage () {
      this.$router.go(-1)
    }
  },
  created () {
    this.getCarts()
  }
}
</script>
