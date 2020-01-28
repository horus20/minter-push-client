<template>
  <div class="container">
    <div class="row header-section">
      <div class="col-2">logo</div>
      <div class="col-7 text-center"><span class="font-weight-bold">#{{ uid }}</span></div>
      <div class="col-3 language-box text-right"><span class="">RU</span></div>
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
                <span v-for="balance in balances">{{ prettyFormat(balance.amount) }} <span class="symbol">{{ balance.coin }}</span></span>
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
          <b-button variant="outline-info" block disabled v-on:click="showEmailTransfer">Отправить на email</b-button>
          <b-button variant="outline-info" block disabled v-on:click="showPushTransfer">Создать новый push-wallet</b-button>
        </div>
      </div>

      <div class="row payment-sections">
        <div class="col hide-md">
          <h4>Потратить</h4>
          <b-button variant="outline-success" block disabled>Оплатить телефон</b-button>
          <b-button variant="outline-secondary" block disabled>Купить Gift карты (yandex, ozon)</b-button>
          <b-button variant="outline-secondary" block disabled> ????? </b-button>
        </div>
      </div>

      <div class="row payment-sections">
        <div class="col hide-md">
          <h4>Узнать больше о ...</h4>
          <a class="btn-outline-info btn btn-sm btn-block" href="https://minterpush.ru" target="_blank">Проект Minter push wallet</a>
          <a class="btn-outline-info btn btn-sm btn-block" href="https://about.minter.network/ru/" target="_blank">о проекте Minter</a>
          <a class="btn-outline-info btn btn-sm btn-block" href="https://ru.minter.wiki/" target="_blank">Wiki проекта</a>
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
          style="max-width: 20rem;"
          class="mb-2"
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
    <b-modal id="modalError" size="sm" centered title="Error" ok-only
             header-bg-variant="danger"
             header-text-variant="light"
    >

      <p>
        {{ errorMsg }}
      </p>
    </b-modal>

    <!-- success -->
    <b-modal id="modalSuccess" size="sm" centered title="Success" ok-only
             header-bg-variant="info"
             header-text-variant="light"
    >

      <p>
        {{ successMsg }}
      </p>
    </b-modal>

    <!-- transfer modal -->
    <b-modal id="modalTransfer" size="sm" centered title="Transfer"
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

        <b-form-group id="input-group-2" label="Сумма перевода:" label-for="input-value">
          <b-form-input
            id="input-value"
            v-model="transfer.value"
            type="number"
            required
            placeholder="Введите сумму"
          ></b-form-input>
        </b-form-group>

        <b-form-group id="input-group-3" label="Монета перевода:" label-for="input-coin">
          <b-form-select v-model="transfer.symbol" required>
            <b-form-select-option v-for="balance in balances" v-bind:value="balance.coin">{{ balance.coin }}</b-form-select-option>
          </b-form-select>
        </b-form-group>
      </b-form>
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

  //const BACKEND_BASE_URL = 'https://minterpush.ru'
  const BACKEND_BASE_URL = 'http://localhost:3048'
  const EXPLORER_BASE_URL = 'https://explorer-api.minter.network'

  export default {
    data () {
      return {
        uid: '',
        pincode: '',

        errorMsg: '',
        successMsg: '',

        isCreateNew: true,
        isLogin: false,

        privateKey: null,
        address: null,

        balances: [],
        nonce: 0,

        transfer: {
          address: '',
          value: '',
          symbol: '',
        }
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
              this.isCreateNew = true;
            } else {
              this.isCreateNew = false;
            }
          }
        } catch (error) {
          this.isCreateNew = false;
          console.error(error);
        }
      },
      /**
       * generate address
       */
      generateAddress: function () {
        // формируем новый mx-адрес из url+pincode
        const privateKey = SHA256(`${this.uid}${this.pincode}`).toString();
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
        try {
          let response = await axios.get(`${EXPLORER_BASE_URL}/api/v1/addresses/${this.address}`)
          if (response.data && response.data.data && response.data.data.balances) {
            this.balances = response.data.data.balances
          }

          response = await axios.get(`${EXPLORER_BASE_URL}/api/v1/addresses/${this.address}/transactions`)
          if (response.data && response.data.meta && response.data.meta) {
            this.nonce = response.data.meta.total
          }
        } catch (error) {
          this.errorMsg = 'Ошибка обновления информации о балансе'
          this.$bvModal.show('modalError')
        }
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
          });
          if (response.status === 201) {
            if (response.data.status === 100) {
              // activate complete!
              this.isCreateNew = false;
              this.isLogin = true;
              this.updateBalance()
              return ;
            }
          }
          // show error
        } catch (error) {
          console.error(error);
        }
        this.errorMsg = 'Ошибка актиации, попробуйте еще раз'
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
          });
          if (response.status === 200) {
            if (response.data.status === 100) {
              // login success
              this.isCreateNew = false;
              this.isLogin = true;
              this.updateBalance()
              return ;
            }
          }
          // show error
        } catch (error) {
          console.error(error);
          this.errorMsg = 'Ошибка авторизации, возможно неверный pincode, попробуйте еще раз'
          this.$bvModal.show('modalError')
        }
      },
      prettyFormat: function(value) {
        return Number(value).toFixed(5)
      },
      checkTransferForm: function () {
        const isValid = this.transfer.address !== "" && this.transfer.value !== "" && this.transfer.symbol !== ""

        if (!isValid) {
          this.errorMsg = 'Ошибка, заполнены не все поля'
          this.$bvModal.show('modalError')
        }
        return isValid
      },
      showTransfer: function () {
        this.$bvModal.show('modalTransfer');
      },
      showPushTransfer: function () {
        this.$bvModal.show('modalTransfer');
      },
      showEmailTransfer: function () {
        this.$bvModal.show('modalTransfer');
      },
      sendTransferModal: async function (bvModalEvt) {
        // Prevent modal from closing
        bvModalEvt.preventDefault()

        if (!this.checkTransferForm()) {
          return
        }

        await this.sendTransfer();

        // Hide the modal manually
        this.$nextTick(() => {
          this.$bvModal.hide('modalTransfer')
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
      sendTransfer: async function () {
        try {
          const txData = new TxDataSend({
            to: toBuffer(this.transfer.address),
            coin: coinToBuffer(this.transfer.symbol),
            value: `0x${convertToPip(this.transfer.value, 'hex')}`,
          })
          const tx = new Tx({
            nonce: String('0x' + this.toHex(this.nonce)),
            chainId: '0x01',
            gasPrice: '0x01',
            gasCoin: coinToBuffer(this.transfer.symbol),
            type: TX_TYPE.SEND,
            data: txData.serialize(),
            signatureType: '0x01'
          })
          const privateKeyBuffer = Buffer.from(this.privateKey, 'hex');
          tx.signatureData = (new TxSignature()).sign(tx.hash(false), privateKeyBuffer).serialize();
          const serializedTx = tx.serialize().toString('hex');

          // send to back
          const txHash = await axios.post(`${BACKEND_BASE_URL}/api/${this.uid}/send`, {
            mxaddress: this.address,
            rawTx: serializedTx,
          });

          this.successMsg = 'Транзакция успешно отправлена'
          this.$bvModal.show('modalSuccess')
          await this.updateBalance();

        } catch (error) {
          console.error(error);
          this.errorMsg = 'Извините при отправке произошла ошибка'
          this.$bvModal.show('modalError')
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
</style>
