<template>
  <div class="container" style="margin-bottom: 30px;">
    <div class="row header-section">
      <div class="col-2"><b-img src="/minter-logo-circle.png" fluid alt="" style="max-width: 36px; margin-top: 3px;"></b-img></div>
      <div class="col-10" style="padding-top: 6px;"><h1 style="font-size: 1.5rem">Minter push wallet</h1></div>
    </div>
    <div class="row">
      <div class="col">
        <h3 style="font-size: 1.2rem">Если у вас есть ссылка перейдите по ней</h3>
        <p class="text-monospace">При первом переходе по ссылке вам будет предложено создать 6ти значный пинкод, после этого произойдет активация кошелька.
          При каждом следующем открытии ссылки с любого устройства вам просто нужно ввести этот пинкод и получить доступ к кошельку.<br>
          Важное уточнение - seed-фраза и приватный ключ никогда не покидают устройства пользователя, подпись транзакций производится на стороне клиента, после активации никто кроме вас не имеет доступа к кошельку.
        </p>
      </div>
    </div>
    <div class="row" style="margin-top: 30px;">
      <div class="col">
        <h3 style="font-size: 1.2rem">Создание нового push wallet</h3>
        <b-button variant="outline-info" v-on:click="createNew">Создать новый push-wallet</b-button>

        <ul v-if="isCreateNew" style="padding-left: 0; list-style: none; margin-top: 10px;">
          <li class="text-monospace">1. Пополните <b-button href="#" variant="outline-success" size="sm" v-on:click="copyToClipboard(mxaddress)">Скопировать</b-button><br><strong>
            <a v-bind:href="linka" target="_blank">{{ mxaddress}}</a></strong><br>
            <div class="text-center"><qrcode v-bind:value="mxaddress" :options="{ width: 200 }"></qrcode></div><br>
            на сумму, которая будет переведена на pushwallet после активации (за вычетом небольшой комиссии)
          </li>
          <li class="text-monospace">2. Отправьте ссылку: <b-button href="#" variant="outline-success" size="sm" v-on:click="copyToClipboard(link)">Скопировать</b-button><br><strong><a v-bind:href="link" target="_blank">{{ link }}</a></strong>
            <div class="text-center"><qrcode v-bind:value="link" :options="{ width: 200 }"></qrcode></div>
          </li>
          <li class="text-monospace">3. При первом открытии этой ссылки нужно будет придумать pincode, который позволит распоряжаться средствами.</li>
        </ul>
      </div>
    </div>
    <div class="row" style="margin-top: 30px;">
      <div class="col">
        <h3 style="font-size: 1.2rem">Расширенный режим создания push wallet</h3>
        <b-button v-if="!isCreateBlockNew" v-b-toggle.createDetail variant="outline-info">Параметры</b-button>

        <b-collapse v-if="!isCreateBlockNew" id="createDetail">
          <div class="row">
            <div class="col">
              <p>Для кого этот интерфейс? Для тех, кто хочет создать блок push-кошельков с одинаковыми стартовыми параметрами.</p>
            </div>
          </div>
          <div class="row">
            <div class="col">
              <b-form ref="form">
                <!--<b-form-group
                  label="Ваш email:"
                  description="Необходим для редактирования"
                >
                  <b-form-input
                    v-model="createParams.email"
                    type="email"
                    placeholder="Введите ваш адрес"
                  ></b-form-input>
                </b-form-group>

                <b-form-group
                  label="Пароль:"
                  description="Необходим для редактирования"
                >
                  <b-form-input
                    v-model="createParams.password"
                    type="text"
                  ></b-form-input>
                </b-form-group>-->

                <b-form-group
                  label="Необходимое количество кошельков:"
                  description="Столько адресов будет создано"
                >
                  <b-form-input
                    v-model="createParams.params.count"
                    type="number" minlength="2" max="99"
                  ></b-form-input>
                </b-form-group>

                <b-form-group
                  label="Сумма активации:"
                  description="Сумма, которая будет зачисляться на счет при активации кошелька"
                >
                  <b-form-input
                    v-model="createParams.params.amount"
                    type="text" minlength="2"
                  ></b-form-input>
                </b-form-group>

                <b-form-group
                  label="Монета:"
                >
                  <b-form-input
                    v-model="createParams.params.symbol"
                    type="text" minlength="2" disabled
                    placeholder="BIP"
                  ></b-form-input>
                </b-form-group>

                <b-form-group
                  label="Текст:"
                  description="Текст, который увидит пользователь после активации кошелька"
                >
                  <b-form-textarea
                    v-model="createParams.params.notice"
                    type="text" minlength="2"
                  ></b-form-textarea>
                </b-form-group>

                <b-button variant="success" block v-on:click="createNewEx">Создать компанию</b-button>
              </b-form>
            </div>
          </div>
        </b-collapse>

        <ul v-if="isCreateBlockNew" style="padding-left: 0; list-style: none; margin-top: 10px;">
          <li class="text-monospace">1. Пополните <b-button href="#" variant="outline-success" size="sm" v-on:click="copyToClipboard(mxaddress)">Скопировать</b-button><br><strong>
            <a v-bind:href="linka" target="_blank">{{ mxaddress}}</a></strong><br>
            <div class="text-center"><qrcode v-bind:value="mxaddress" :options="{ width: 200 }"></qrcode></div><br>
            на сумму <b-button href="#" variant="outline-success" size="sm" v-on:click="copyToClipboard(activateSum)">{{ activateSum }}</b-button> (сумма активации для всех кошельков + небольшая комиссия)
          </li>
          <li class="text-monospace">2. Cписок ссылок: <b-button href="#" variant="outline-success" size="sm" v-on:click="copyToClipboard(linksString)">Скопировать</b-button>
            <ul style="padding-left: 5px; list-style: none;">
              <li v-for="link in links"><a v-bind:href="link" target="_blank">{{ link }}</a></li>
            </ul>
          </li>
          <li class="text-monospace">3. Передайте эти ссылки, при первом открытии каждой будет происходить активация с начислением стартового баланса.</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  import VueQrcode from '@chenfengyuan/vue-qrcode'
  import BigNumber from 'bignumber.js'
  import Vue from 'vue'
  import VueClipboard from 'vue-clipboard2'

  if (process.client) {
    Vue.use(VueClipboard)
  }

  const BACKEND_BASE_URL = 'https://minterpush.ru'
  //const BACKEND_BASE_URL = 'http://localhost:3048'
  const LINK = 'https://minterpush.ru/w/'
  const ACTIVATE_FEE = '0.1'    // todo: change this place

  export default {
    components: {
      qrcode: VueQrcode,
    },
    data () {
      return {
        mxaddress: '',
        link: '',
        links: [],
        isCreateNew: false,
        isCreateBlockNew: false,
        activateSum: '',

        createParams: {
          email: '',
          password: '',
          params: {
            count: 1,
            amount: '0',
            symbol: 'BIP',
            title: '',
            notice: ''
          }
        }
      }
    },
    computed: {
      linka() {
        return `https://explorer.minter.network/address/${this.mxaddress}`
      },
      linksString() {
        return this.links.join(' ; ')
      }
    },
    methods: {
      createNew: async function () {
        try {
          const response = await axios.post(`${BACKEND_BASE_URL}/api/company`, {})
          if (response && response.status === 201) {
            if (response.data.status === 100) { // new
              // show instruction's
              this.isCreateNew = true
              this.mxaddress = response.data.warehouseWallet.mxaddress
              this.link = LINK + response.data.wallets[0].uid
            } else {
              this.isCreateNew = false
            }
          }
        } catch (error) {
          this.isCreateNew = false
          console.error(error)
        }
      },
      createNewEx: async function () {
        try {
          let activateSum = new BigNumber(0)
          const activateAmount = new BigNumber(this.createParams.params.amount ?? 0)
          if (this.createParams.params.count > 1 && activateAmount.gt(0)) {
            activateSum = activateAmount.plus(ACTIVATE_FEE).multipliedBy(this.createParams.params.count)
          }
          this.activateSum = activateSum.toNumber().toFixed(5) + ' ' + this.createParams.params.symbol

          const response = await axios.post(`${BACKEND_BASE_URL}/api/company`, this.createParams)
          if (response && response.status === 201) {
            if (response.data.status === 100) { // new
              // show instruction's
              this.isCreateBlockNew = true
              this.mxaddress = response.data.warehouseWallet.mxaddress
              this.links = response.data.wallets.map(({ uid }) => `${LINK}${uid}`)
            } else {
              this.isCreateNew = false
            }
          }
        } catch (error) {
          this.isCreateNew = false
          console.error(error)
        }
      },
      copyToClipboard: function (message) {
        this.$copyText(message).then(function (e) {
        }, function (e) {
        })
        return false;
      }
    }
  }
</script>

<style scoped>

</style>
