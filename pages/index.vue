<template>
  <div class="container">
    <div class="row">
      <div class="col">
        <h1>Minter push wallet</h1>
        <h3>Если у вас есть ссылка перейдте по ней</h3>
      </div>
    </div>
    <div class="row">
      <div class="col">
        <h3>Создание нового push wallet</h3>
        <b-button variant="outline-info" v-on:click="createNew">Создать новый push-wallet</b-button>

        <ul v-if="isCreateNew" style="padding-left: 0; list-style: none;">
          <li>1. Пополните <br><strong>
            <a v-bind:href="linka" target="_blank">{{ mxaddress}}</a></strong><br> на сумму, которая будет переведена на pushwallet после активации (за вычетом небольшой комиссии)</li>
          <li>2. Отправьте ссылку: <br><strong><a v-bind:href="link" target="_blank">{{ link }}</a></strong></li>
          <li>3. При первом открытии этой ссылки нужно будет придумать pincode, который позволит распоряжаться средствами.</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'

  //const BACKEND_BASE_URL = 'https://minterpush.ru'
  const BACKEND_BASE_URL = 'http://localhost:3048'
  const LINK = 'https://minterpush.ru/w/';

  export default {
    data () {
      return {
        mxaddress: '',
        link: '',
        isCreateNew: false,
      }
    },
    computed: {
      linka() {
        return `https://explorer.minter.network/address/${this.mxaddress}`
      }
    },
    methods: {
      createNew: async function () {
        try {
          const response = await axios.post(`${BACKEND_BASE_URL}/api/company`, {})
          if (response && response.status === 201) {
            if (response.data.status === 100) { // new
              // show "hi form" and create pincode
              this.isCreateNew = true;
              this.mxaddress = response.data.warehouseWallet.mxaddress;
              this.link = LINK + response.data.wallets[0].uid
            } else {
              this.isCreateNew = false;
            }
          }
        } catch (error) {
          this.isCreateNew = false;
          console.error(error);
        }
      }
    }
  }
</script>

<style scoped>

</style>
