<template>
  <div class="container">
    <div class="row header-section">
      <div class="col-2"><b-img src="/minter-logo-circle.png" fluid alt="" style="max-width: 36px; margin-top: 3px;"></b-img></div>
      <div class="col-7 text-center" style="padding-top: 8px;"><span class="font-weight-bold">#{{ uid }}</span></div>
      <div class="col-3 language-box text-right" style="padding-top: 8px;"><span class="">RU</span></div>
    </div>

    <template v-if="isLogin">
      <!-- Section balance -->
      <div class="row balance-section">
        <div class="col">
          <div class="card">
            <div class="card-body">
              <h5 class="card-title">Баланс:
                <!-- <span class="balance-usd">~ 10.00 $</span><span class="balance-fiat">(620.00 руб)</span>-->
              </h5>
              <div>
                <span v-for="balance in balances">{{ prettyFormat(balance.amount) }} <span class="symbol">{{ balance.coin }}</span> <span v-if="balance.usdAmount > 0">(${{ balance.usdAmount }})</span></span>
              </div>
              <div class="row">
                <div class="col">
                  <b-button variant="outline-success" block size="sm" v-on:click="updateBalance">Обновить данные баланса</b-button>
                </div>
              </div>
              <!--<div class="row">
                <div class="col">
                  <b-button v-b-toggle.balanceDetail variant="info" size="sm">Показать детальную информацию о балансе</b-button>
                </div>
              </div>

              <b-collapse id="balanceDetail">
                <div class="row balances">
                  <div class="col">
                    <span v-for="balance in balances">{{ prettyFormat(balance.amount) }} <span class="symbol">{{ balance.coin }}</span></span>
                  </div>
                </div>
              </b-collapse>-->

            </div>
          </div>
        </div>
      </div>
      <!-- end section -->

      <div class="row transfer-section">
        <div class="col hide-md">
          <h4>Перевести</h4>
          <b-button variant="outline-info" block v-on:click="showTransfer">Другой кошелек в сети Minter</b-button>
          <!-- <b-button variant="outline-info" block disabled v-on:click="showEmailTransfer">Отправить на email</b-button>-->
          <b-button variant="outline-info" block v-on:click="showPushTransfer">Создать новый push-wallet</b-button>
          <b-button variant="outline-info" block v-on:click="showMyAddress">Показать мой адрес для переводов</b-button>
        </div>
      </div>

      <div class="row payment-sections">
        <div class="col hide-md">
          <h4>Потратить</h4>
          <b-button variant="outline-success" block v-on:click="showPhoneUp">Оплатить телефон</b-button>
          <b-button variant="outline-success" block v-on:click="showTimeLoop">Пополнить счет в игре TimeLoop</b-button>
          <b-button variant="outline-secondary" block disabled>Купить Gift карты (yandex, ozon)</b-button>
          <b-button variant="outline-secondary" block disabled> ????? </b-button>
        </div>
      </div>

      <div class="row payment-sections">
        <div class="col hide-md">
          <h4>Узнать больше ...</h4>
          <!--<a class="btn-outline-info btn btn-sm btn-block" href="https://minterpush.ru" target="_blank">Проект Minter push wallet</a>-->
          <a class="btn-outline-info btn btn-sm btn-block" href="https://about.minter.network/ru/" target="_blank">о проекте Minter</a>
          <a class="btn-outline-info btn btn-sm btn-block" href="https://ru.minter.wiki/" target="_blank">Minter wiki</a>
        </div>
      </div>
    </template>
    <template v-else>
      <!-- section greeting -->
      <template v-if="isCreateNew">
        <b-card
          title="Новый кошелёк"
        >
          <b-card-text>
            Привет, это новый push-кошелёк, чтобы начать пользоваться придумай pincode из 6 цифр. Всё просто доступ к кошельку будет только у того кто знает ссылку и pincode.
          </b-card-text>

          <input v-model="pincode" type="number" minlength="6" max="999999" class="form-control pincode">
          <b-button variant="success" block :disabled="isPincodeValid" v-on:click="activate">Получить доступ</b-button>
        </b-card>
      </template>
      <template v-else>
        <b-card
          title="Кошелёк"
        >
          <b-card-text>
            Привет, этот push-кошелёк уже ранее был активирован, чтобы получить доступ введи pincode (6 цифр).
          </b-card-text>

          <input v-model="pincode" type="number" minlength="6" max="999999" class="form-control pincode">
          <b-button variant="success" block :disabled="isPincodeValid" v-on:click="login">Получить доступ</b-button>
        </b-card>
      </template>
    </template>

    <!-- error modal -->
    <b-modal id="modalError" centered title="Ошибка" ok-only
             header-bg-variant="danger"
             header-text-variant="light"
    >
      <div v-html="errorMsg">
      </div>
    </b-modal>

    <!-- success -->
    <b-modal id="modalSuccess" centered v-bind:title="successTitle" ok-only
             header-bg-variant="info"
             header-text-variant="light"
    >
      <div v-html="successMsg">
      </div>
      <div v-if="successMsgLink!==''" class="text-center">
        <!-- <b-button href="#" variant="outline-success" size="sm" v-on:click="copyToClipboard(successMsgLink)">Скопировать</b-button><br>-->
        <qrcode v-bind:value="successMsgLink" :options="{ width: 200 }"></qrcode>
      </div>
    </b-modal>

    <!-- transfer modal -->
    <b-modal id="modalTransfer" centered title="Перевод"
             header-bg-variant="info"
             header-text-variant="light"
             @show="resetTransferModal"
             @hidden="resetTransferModal"
             @ok="sendTransferModal"
    >

      <b-form ref="form">
        <b-form-group
          id="input-group-1"
          label="Mx address:"
          label-for="input-mxaddress"
        >
          <b-form-input
            id="input-mxaddress"
            v-model="transfer.address"
            type="text"
            required
            placeholder="Введите адрес Mx...."
          ></b-form-input>
        </b-form-group>

        <b-form-group id="input-group-2" label="Сумма перевода:" label-for="input-value" style="position: relative;">
          <b-form-input
            id="input-value"
            v-model="transfer.value"
            type="number"
            required
            placeholder="Введите сумму"
          ></b-form-input>
          <b-button href="#" variant="danger" size="sm" v-on:click="useMax(1)" class="max-btn">max</b-button>
        </b-form-group>

        <b-form-group id="input-group-3" label="Монета перевода:" label-for="input-coin">
          <b-form-select v-model="transfer.symbol" required>
            <b-form-select-option v-for="balance in balances" v-bind:value="balance.coin">{{ balance.coin }}</b-form-select-option>
          </b-form-select>
        </b-form-group>
      </b-form>
    </b-modal>

    <!-- create new push wallet -->
    <b-modal id="modalPushTransfer" centered title="Создать новый push wallet"
             header-bg-variant="info"
             header-text-variant="light"
             @show="resetTransferModal"
             @hidden="resetTransferModal"
             @ok="sendPushTransferModal"
    >

      <b-form ref="form">
        <b-form-group label="Сумма пополнения:" label-for="input-value"
                      description="Сумма, которая будет зачислена за вычетом небольшой комиссии"
                      style="position: relative;"
        >
          <b-form-input
            id="input-value"
            v-model="transfer.value"
            type="number"
            required
            placeholder="Введите сумму"
            help
          ></b-form-input>
          <b-button href="#" variant="danger" size="sm" v-on:click="useMax(2)" class="max-btn">max</b-button>
        </b-form-group>

        <b-form-group label="Монета:" label-for="input-coin" id="input-group-tp3">
          <b-form-select v-model="transfer.symbol" required>
            <b-form-select-option v-for="balance in balances" v-bind:value="balance.coin">{{ balance.coin }}</b-form-select-option>
          </b-form-select>
        </b-form-group>
      </b-form>
    </b-modal>

    <!-- phone up form -->
    <b-modal id="modalPhoneUpForm" centered title="Пополнить счет"
             header-bg-variant="info"
             header-text-variant="light"
             @show="resetTransferModal"
             @hidden="resetTransferModal"
             @ok="sendPhoneUpModal"
    >

      <b-form ref="form">
        <div class="row">
          <div class="col">
            <p>Пополнение телефона произодится через сторонний сервис <a href="http://biptophone.ru" target="_blank">BipToPhone</a></p>
          </div>
        </div>
        <b-form-group
          id="input-group-pr-1"
          label="Ваш телефон:"
          label-for="input-mxaddress"
        >
            <b-form-input
              id="input-phone"
              v-model="transfer.address"
              type="text"
              required
              placeholder="Телефон +79..."
            ></b-form-input>
        </b-form-group>

        <b-form-group label="Сумма пополнения:" label-for="input-value"
                      description="Сумма, которая будет передана партнеру для зачисления"
                      style="position: relative;"
        >
          <b-form-input
            id="input-value"
            v-model="transfer.value"
            type="number"
            required
            placeholder="Введите сумму"
            help
          ></b-form-input>
          <b-button href="#" variant="danger" size="sm" v-on:click="useMax(3)" class="max-btn">max</b-button>
        </b-form-group>

        <b-form-group label="Монета:" label-for="input-coin" id="input-group-pr3">
          <b-form-select v-model="transfer.symbol" required>
            <b-form-select-option v-for="balance in balances" v-bind:value="balance.coin">{{ balance.coin }}</b-form-select-option>
          </b-form-select>
        </b-form-group>
      </b-form>
    </b-modal>

    <!-- time loopform -->
    <b-modal id="modalTimeloop" centered title="Пополнить счет в TimeLoop"
             header-bg-variant="info"
             header-text-variant="light"
             @show="resetTransferModal"
             @hidden="resetTransferModal"
             @ok="sendTimeLoopModal"
    >

      <b-form ref="form">
        <div class="row">
          <div class="col">
            <p>Пополнение счета игрока в игре <a href="https://timeloop.games/" target="_blank">TimeLoop</a></p>
          </div>
        </div>
        <b-form-group label="Сумма пополнения:" label-for="input-value"
                      description="Сумма, которая будет зачислена на игровой счет"
                      style="position: relative;"
        >
          <b-form-input
            id="input-value"
            v-model="transfer.value"
            type="number"
            required
            placeholder="Введите сумму"
            help
          ></b-form-input>
          <b-button href="#" variant="danger" size="sm" v-on:click="useMax(4)" class="max-btn">max</b-button>
        </b-form-group>

        <b-form-group label="Монета:" label-for="input-coin" id="input-group-tl3">
          <b-form-select v-model="transfer.symbol" required>
            <b-form-select-option v-for="balance in balances" v-bind:value="balance.coin">{{ balance.coin }}</b-form-select-option>
          </b-form-select>
        </b-form-group>
      </b-form>
    </b-modal>

    <b-modal id="loader" class="loader-modal" centered
             :hide-header=true
             :hide-footer=true
             :no-close-on-backdrop=true
             :no-close-on-esc=true
             content-class="loader-modal"
    >
      <div>
        <div class="d-flex justify-content-center">
          <b-spinner class="" label="Loading..." variant="light"></b-spinner>
        </div>
      </div>
    </b-modal>

  </div>
</template>

<script>
  import axios from 'axios'
  import { walletFromPrivateKey } from 'minterjs-wallet'
  import { SHA256 } from 'crypto-js'
  import { TX_TYPE, Tx } from 'minterjs-tx'
  import { toBuffer, convertToPip } from 'minterjs-util'
  import TxDataSend from 'minterjs-tx/src/tx-data/send'
  import { coinToBuffer } from 'minterjs-tx/src/helpers'
  import TxSignature from 'minterjs-tx/src/tx-signature'
  import VueQrcode from '@chenfengyuan/vue-qrcode'
  import * as cryptoRandomString from 'crypto-random-string'
  import BigNumber from 'bignumber.js'
  import Vue from 'vue'
  import VueClipboard from 'vue-clipboard2'

  if (process.client) {
    Vue.use(VueClipboard)
  }

  //const BACKEND_BASE_URL = 'https://minterpush.ru'
  const BACKEND_BASE_URL = 'http://localhost:3048'
  const EXPLORER_BASE_URL = 'https://explorer-api.minter.network'
  const EXPLORER_GATE_API_URL = 'https://gate-api.minter.network'
  const LINK = 'https://minterpush.ru/w/'
  const COURCE_BIP_URL = 'https://api.bip.dev/api/price'
  const COURCE_FIAT_URL = 'https://www.cbr-xml-daily.ru/daily_json.js'

  export default {
    components: {
      qrcode: VueQrcode,
    },
    data () {
      return {
        uid: '',
        pincode: '',

        errorMsg: '',
        successTitle: '',
        successMsg: '',
        successMsgLink: '',

        isCreateNew: true,
        isLogin: false,

        privateKey: null,
        address: null,

        bipToUSD: 0,
        balances: [],
        nonce: 0,

        transfer: {
          address: '',
          value: '',
          symbol: '',
        },

        activateParams: {}
      }
    },
    computed: {
      isPincodeValid() {
        return this.pincode.length !== 6
      }
    },
    created () {
      this.uid = this.$route.params.id
      this.checkAuth()
    },
    mounted() {
      // check
    },
    methods: {
      /**
       * check wallet auth
       * @returns {Promise<void>}
       */
      checkAuth: async function () {
        try {
          const response = await axios.get(`${BACKEND_BASE_URL}/api/${this.uid}`)
          if (response && response.status === 200) {
            if (response.data.status === 50) { // new
              // show "hi form" and create pincode
              this.isCreateNew = true
            } else {
              this.isCreateNew = false
            }
          }
        } catch (error) {
          this.isCreateNew = false
          console.error(error)
        }
      },
      /**
       * generate address
       */
      generateAddress: function () {
        // формируем новый mx-адрес из url+pincode
        const privateKey = SHA256(`${this.uid}${this.pincode}`).toString()
        const privateKeyBuffer = Buffer.from(privateKey, 'hex')
        const wallet = walletFromPrivateKey(privateKeyBuffer)

        // strore param's
        this.address = wallet.getAddressString()
        this.privateKey = privateKey
      },
      /**
       * update wallet balance
       * @returns {Promise<void>}
       */
      updateBalance: async function () {
        this.$bvModal.show('loader')
        try {
          let response;
          response = await axios.get(COURCE_BIP_URL)
          if (response.data && response.data.data && response.data.data.price) {
            this.bipToUSD = new BigNumber(response.data.data.price).div(10000)
          }

          response = await axios.get(`${EXPLORER_BASE_URL}/api/v1/addresses/${this.address}`)
          if (response.data && response.data.data && response.data.data.balances) {
            this.balances = response.data.data.balances.map(({ coin, amount}) => {
              let usdAmount = new BigNumber(0)
              if (coin === 'BIP') {
                usdAmount = new BigNumber(amount).multipliedBy(this.bipToUSD)
              }

              return {
                coin,
                amount,
                usdAmount: usdAmount.toNumber().toFixed(2)
              }
            })
          }

          response = await axios.get(`${EXPLORER_GATE_API_URL}/api/v1/nonce/${this.address}`)
          if (response.data && response.data.data && response.data.data.nonce) {
            this.nonce = Number(response.data.data.nonce) + 1
          }
        } catch (error) {
          this.errorMsg = 'Ошибка обновления информации о балансе'
          this.$bvModal.show('modalError')
        }
        this.$bvModal.hide('loader')
      },
      /**
       * activate wallet
       * @returns {Promise<void>}
       */
      activate: async function () {
        try {
          this.generateAddress()

          // try activate
          const response = await axios.post(`${BACKEND_BASE_URL}/api/${this.uid}`, {
            mxaddress: this.address,
          })
          if (response.status === 200) {
            if (response.data.status === 100) {
              // activate complete!
              const afterActivateResponse = await axios.post(`${BACKEND_BASE_URL}/api/${this.uid}/after`, {
                mxaddress: this.address,
              })
              // wait 6 sec
              await this.sleep(6 * 1000);

              if (afterActivateResponse.status === 200 && afterActivateResponse.data) {
                this.activateParams = afterActivateResponse.data

                if (this.activateParams.title || this.activateParams.notice) {
                  this.successTitle = this.activateParams.title
                  this.successMsg = this.activateParams.notice
                  this.$bvModal.show('modalSuccess')
                }
              }

              this.isCreateNew = false
              this.isLogin = true
              this.updateBalance()
              return
            }
          }
          // show error
        } catch (error) {
          console.error(error)
        }
        this.errorMsg = 'Ошибка актиации, ваш счет еще пуст. Попробуйте позже.'
        this.$bvModal.show('modalError')
      },
      /**
       * try login with pincode
       * @returns {Promise<void>}
       */
      login: async function () {
        try {
          this.generateAddress()

          // try activate
          const response = await axios.post(`${BACKEND_BASE_URL}/api/${this.uid}`, {
            mxaddress: this.address,
          })
          if (response.status === 200) {
            if (response.data.status === 100) {
              // login success
              this.isCreateNew = false
              this.isLogin = true
              this.updateBalance()
              return
            }
          }
          // show error
        } catch (error) {
          console.error(error)
          this.errorMsg = 'Ошибка авторизации, возможно неверный pincode, попробуйте еще раз'
          this.$bvModal.show('modalError')
        }
      },
      /**
       * @param value
       * @returns {string}
       */
      prettyFormat: function(value) {
        return Number(value).toFixed(5)
      },
      /**
       * @returns {boolean}
       */
      checkTransferForm: function () {
        const isValid = this.transfer.address !== "" && this.transfer.value !== "" && this.transfer.symbol !== ""

        if (!isValid) {
          this.errorMsg = 'Ошибка, заполнены не все поля'
          this.$bvModal.show('modalError')
        }
        return isValid
      },
      showTransfer: function () {
        this.$bvModal.show('modalTransfer')
      },
      showPushTransfer: function () {
        this.$bvModal.show('modalPushTransfer')
      },
      showEmailTransfer: function () {
        this.$bvModal.show('modalTransfer')
      },
      showPhoneUp: function () {
        this.$bvModal.show('modalPhoneUpForm')
      },
      showTimeLoop: function () {
        this.$bvModal.show('modalTimeloop')
      },
      showMyAddress: function () {
        this.successMsg = `Ваш адрес для приёма платежей! <br>
                    <strong>${this.address}</strong>`
        this.successMsgLink = this.address
        this.$bvModal.show('modalSuccess')
      },
      sendTransferModal: async function (bvModalEvt) {
        // Prevent modal from closing
        bvModalEvt.preventDefault()

        if (!this.checkTransferForm()) {
          return
        }

        const result = await this.sendTransfer(this.transfer.address, this.transfer.value, this.transfer.symbol)
        if (result) {
          this.successMsg = 'Транзакция успешно отправлена'
          this.$bvModal.show('modalSuccess')
        }

        // Hide the modal manually
        this.$nextTick(() => {
          this.$bvModal.hide('modalTransfer')
        })
      },
      sendPushTransferModal: async function (bvModalEvt) {
        // Prevent modal from closing
        bvModalEvt.preventDefault()

        this.transfer.address = '0x0'
        if (!this.checkTransferForm()) {
          return
        }

        try {
          const response = await axios.post(`${BACKEND_BASE_URL}/api/company`, {})
          if (response && response.status === 201) {
            if (response.data.status === 100) { // new
              const mxaddress = response.data.warehouseWallet.mxaddress
              const link = LINK + response.data.wallets[0].uid

              const result = await this.sendTransfer(mxaddress, this.transfer.value, this.transfer.symbol)
              if (result) {
                this.successMsg = `Кошелёк успешно создан! <br>
                    Ссылка: <strong><a href="${link}" target="_blank">${link}</a></strong>`
                this.successMsgLink = link
                this.$bvModal.show('modalSuccess')
              }

            } else {
              throw new Error(response.data)
            }
          } else {
            throw new Error(response.status)
          }
        } catch (error) {
          console.error(error)
          this.errorMsg = 'Извините при отправке произошла ошибка'
          this.$bvModal.show('modalError')
        }

        // Hide the modal manually
        this.$nextTick(() => {
          this.$bvModal.hide('modalPushTransfer')
        })
      },
      sendPhoneUpModal: async function (bvModalEvt) {
        bvModalEvt.preventDefault()

        if (!this.checkTransferForm()) {
          return
        }

        try {
          const response = await axios.post(`${BACKEND_BASE_URL}/api/${this.uid}/services/phone`, {
            mxaddress: this.address,
            phone: this.transfer.address,
          })
          if (response && response.status === 200) {
            if (response.data) {
              const mxaddress = response.data

              const result = await this.sendTransfer(mxaddress, this.transfer.value, this.transfer.symbol)
              if (result) {
                this.successMsg = `Средства отправлены партнеру и скоро будут зачислены на указанный (${this.transfer.address}) номер телефона!`
                this.successMsgLink = ''
                this.$bvModal.show('modalSuccess')
              }

            } else {
              this.errorMsg = 'Наш партнер не смог проверить указанный номер телефона'
              this.$bvModal.show('modalError')
            }
          } else {
            throw new Error(response.status)
          }
        } catch (error) {
          console.error(error)
          this.errorMsg = 'Извините при отправке произошла ошибка'
          this.$bvModal.show('modalError')
        }

        // Hide the modal manually
        this.$nextTick(() => {
          this.$bvModal.hide('modalPhoneUpForm')
        })
      },
      sendTimeLoopModal: async function (bvModalEvt) {
        bvModalEvt.preventDefault()

        this.transfer.address = '0x0'
        if (!this.checkTransferForm()) {
          return
        }
        // create timeloop link
        const secretCode = cryptoRandomString({
          length: 10,
          type: 'hex',
        })
        const secretCodeHash = SHA256(secretCode).toString()
        const timeloopAddress = 'Mx3650064486380210127159872871912061022891'
        const link = `https://timeloop.games/?gift=${secretCode}`
        const result = await this.sendTransfer(timeloopAddress, this.transfer.value, this.transfer.symbol, secretCodeHash)
        if (result) {
          this.successMsg = `Остался последний шаг, перейди по ссылке! <br>
                    Ссылка: <strong><a href="${link}" target="_blank">${link}</a></strong>`
          this.successMsgLink = link
          this.$bvModal.show('modalSuccess')
        }

        this.$nextTick(() => {
          this.$bvModal.hide('modalTimeloop')
        })
      },
      resetTransferModal: function () {
        this.transfer = {
          address: '',
          value: '',
          symbol: '',
        }
      },
      toHex: function (d) {
        return  ("0"+(Number(d).toString(16))).slice(-2).toUpperCase()
      },
      /**
       *
       * @returns {Promise<void>}
       */
      sendTransfer: async function (to, value, symbol, payload = null) {
        try {
          const txData = new TxDataSend({
            to: toBuffer(to),
            coin: coinToBuffer(symbol),
            value: `0x${convertToPip(value, 'hex')}`,
          })
          const txParams = {
            nonce: String('0x' + this.toHex(this.nonce)),
            chainId: '0x01',
            gasPrice: '0x01',
            gasCoin: coinToBuffer(symbol),
            type: TX_TYPE.SEND,
            data: txData.serialize(),
            signatureType: '0x01'
          }
          if (payload) {
            txParams.payload = `0x${Buffer.from(payload, 'utf-8').toString('hex')}`
          }

          const tx = new Tx(txParams)
          const privateKeyBuffer = Buffer.from(this.privateKey, 'hex')
          tx.signatureData = (new TxSignature()).sign(tx.hash(false), privateKeyBuffer).serialize()
          const serializedTx = tx.serialize().toString('hex')

          // send to back
          const txHash = await axios.post(`${BACKEND_BASE_URL}/api/${this.uid}/send`, {
            mxaddress: this.address,
            rawTx: serializedTx,
          })

          await this.sleep(2 * 1000)
          await this.updateBalance()

          return true
        } catch (error) {
          console.error(error)
          this.errorMsg = 'Извините при отправке произошла ошибка'
          this.$bvModal.show('modalError')

          return false
        }
      },
      sleep: async function (ms) {
        this.$bvModal.show('loader')
        await new Promise(resolve => setTimeout(resolve, ms))
        this.$bvModal.hide('loader');
      },
      copyToClipboard: function (message) {
        this.$copyText(message).then(function (e) {
        }, function (e) {
          console.log(message, e)
        })
      },
      useMax: function (type = 1) {
        if (type === 1) {// simple transfer
          this.transfer.value = new BigNumber(this.balances[0].amount).minus(0.01).toString()
        }
        if (type === 2) {// pushwallet
          this.transfer.value = new BigNumber(this.balances[0].amount).minus(0.11).toString()
        }
        if (type === 3) {// phone
          this.transfer.value = new BigNumber(this.balances[0].amount).minus(0.01).toString()
        }
        if (type === 4) {// timeloop
          this.transfer.value = new BigNumber(this.balances[0].amount).minus(0.138).toString()
        }
      }
    }
  }
</script>

<style>
  .header-section {
    margin-bottom: 5px;
  }
  .balances {
    margin-top: 5px;
  }
  .balance-usd {
    font-size: 16pt;
  }
  .balance-fiat {
    font-size: 12pt;
  }
  .balance-token {
    margin-right: 10px;
  }
  .language-box {
    font-size: 10pt;
  }
  .balance-section {
    margin-bottom: 20px;
  }
  .transfer-section {
    margin-bottom: 20px;
  }
  .payment-sections {
    margin-bottom: 20px;
  }
  .pincode {
    margin-bottom: 10px;

  }
  .loader-modal {
    background: transparent;
    border: none;
  }
  .max-btn {
    float: right;
    position: absolute;
    right: 4px;
    top: 38px;
    padding: 2px 10px;
  }
</style>
