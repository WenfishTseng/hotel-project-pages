<template>
  <nav class="navbar sticky-top navbar-expand-lg navbar-light bg-light py-3">
    <div class="container">
      <a class="navbar-brand" href="#">Hotel 543</a>
      <button
        class="navbar-toggler"
        type="button"
        data-bs-toggle="collapse"
        data-bs-target="#navbarSupportedContent"
        aria-controls="navbarSupportedContent"
        aria-expanded="false"
        aria-label="Toggle navigation"
      >
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav me-auto mb-2 mb-lg-0">
          <li class="nav-item">
            <router-link class="nav-link" to="/products">產品列表</router-link>
          </li>
          <li class="nav-item">
            <router-link class="nav-link" to="/cart">購物車</router-link>
          </li>
          <li class="nav-item">
            <router-link class="nav-link" to="/login">登入</router-link>
          </li>
        </ul>
        <router-link class="nav-link" to="/cart">
          <button class="btn btn-outline-dark position-relative">
            <i class="fa-solid fa-cart-shopping"></i>
            <span
              class="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-danger"
            >
              {{ cartData?.carts?.length }}
            </span>
          </button>
        </router-link>
      </div>
    </div>
  </nav>
  <router-view />
</template>

<script>
import emitter from '@/libs/emitter.js'

export default {
  data () {
    return {
      // 購物車列表
      qty: 1,
      cartData: {}
    }
  },
  methods: {
    getCarts () {
      this.$http
        .get(`${process.env.VUE_APP_API}/api/${process.env.VUE_APP_PATH}/cart`)
        .then((response) => {
          this.cartData = response.data.data
        })
        .catch((error) => {
          alert(error)
        })
    }
  },
  created () {
    this.getCarts()
    emitter.on('get-cart', () => {
      this.getCarts()
    })
  }
}
</script>
