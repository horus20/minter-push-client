<template>
  <div class="container">
    <div class="row header-section">
      <div class="col-2"><b-img src="/minter-logo-circle.png" fluid alt="" style="max-width: 36px; margin-top: 3px;"></b-img></div>
      <div class="col-10"><h1 style="font-size: 1.5rem">Minter push wallet</h1></div>
    </div>
    <div class="row">
      <div class="col">
        <h3 style="font-size: 1.2rem">Если у вас есть ссылка перейдите по ней</h3>
        <p class="text-monospace">При первом переходе по ссылке вам будет предложено создать 6ти значный пинкод, после этого произойдет активация кошелька.
          При каждом следующем открытии ссылки с любого устройства вам просто нужно ввести этот пинкод и получить доступ к кошельку.<br>
          Важное уточнение - seed-фраза и приватный ключ никогда не покидают устройства пользователя, подпись транзакций производится на сторое клиента, после активации никто кроме вас не имеет доступа к кошельку.
        </p>
      </div>
    </div>
    <div class="row" style="margin-top: 30px;">
      <div class="col">
        <h3 style="font-size: 1.2rem">Создание нового push wallet</h3>
        <b-button variant="outline-info" v-on:click="createNew">Создать новый push-wallet</b-button>

        <ul v-if="isCreateNew" style="padding-left: 0; list-style: none; margin-top: 10px;">
          <li class="text-monospace">1. Пополните <br><strong>
            <a v-bind:href="linka" target="_blank">{{ mxaddress}}</a></strong><br>
            <div class="text-center"><qrcode v-bind:value="mxaddress" :options="{ width: 200 }"></qrcode></div><br>
            на сумму, которая будет переведена на pushwallet после активации (за вычетом небольшой комиссии)
          </li>
          <li class="text-monospace">2. Отправьте ссылку: <br><strong><a v-bind:href="link" target="_blank">{{ link }}</a></strong>
            <div class="text-center"><qrcode v-bind:value="link" :options="{ width: 200 }"></qrcode></div>
          </li>
          <li class="text-monospace">3. При первом открытии этой ссылки нужно будет придумать pincode, который позволит распоряжаться средствами.</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  import VueQrcode from '@chenfengyuan/vue-qrcode'

  const BACKEND_BASE_URL = 'https://minterpush.ru'
  //const BACKEND_BASE_URL = 'http://localhost:3048'
  const LINK = 'https://minterpush.ru/w/';

  export default {
    components: {
      qrcode: VueQrcode
    },
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
              // show instruction's
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
