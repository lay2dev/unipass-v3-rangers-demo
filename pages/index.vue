<template>
  <div id="page-demo" class="unipass-page">
    <i class="background-logo iconfont icon-logo"></i>
    <div class="head">UniPass Demo</div>
    <div v-if="username">
      <div>
        <br />
        <h3>{{ username }}</h3>
        <br />
      </div>
      <el-button class="transfer" type="primary" @click="logout"
        >logout</el-button
      >
    </div>
    <div v-else>
      <div>entry: {{ config.entry }}</div>

      <br />
      <el-radio v-model="toTheme" label="dark">Dark</el-radio>
      <el-radio v-model="toTheme" label="light">Light</el-radio>
      <el-button type="primary" class="transfer login" @click="connect">
        login
      </el-button>
    </div>
    <el-tabs
      v-show="username"
      v-model="activeTab"
      class="body"
      type="border-card"
    >
      <el-tab-pane label="RPG Transaction" name="sign_transaction">
        <el-form
          ref="form"
          class="body-input"
          label-position="top"
          :model="form"
          @submit.native.prevent
        >
          <el-form-item label="Your Address:" prop="address">
            <template #label>
              <span>Your Address:</span>
              <i
                v-show="myAddress"
                class="iconfont icon-copy sea-background"
                @click="bindCopy"
              ></i>
            </template>
            <el-input
              v-model="myAddress"
              disabled
              readonly
              type="textarea"
              resize="none"
              :autosize="{ minRows: 1 }"
            />
          </el-form-item>
          <span>
            <b style="color: black">
              <a href="https://robin-faucet.rangersprotocol.com">
                Rangers Faucet:
              </a>
            </b>
          </span>
          <el-form-item label="Token Type:" prop="address">
            <el-radio-group v-model="tokenType">
              <el-radio-button label="RPG"></el-radio-button>
              <el-radio-button label="DAI(ERC20)"></el-radio-button>
            </el-radio-group>
          </el-form-item>

          <el-form-item label="Your Balance:" prop="address">
            <el-input v-model="myBalanceFormat" disabled readonly />
          </el-form-item>
          <el-form-item label="Transfer To:" prop="address">
            <el-input v-model="toAddress" clearable @blur="onAddressChanged" />
          </el-form-item>
          <el-form-item label="Amount:" prop="address">
            <el-input v-model="toAmount" clearable />
          </el-form-item>
          <el-form-item label="Fee(RPG):" prop="fee">
            <el-input v-model="toFeeAmount" clearable />
          </el-form-item>
          <el-form-item label="Description:" prop="description">
            <el-input v-model="toDescription" clearable />
          </el-form-item>
        </el-form>

        <br />
        <div v-if="tokenType === 'RPG'">
          <el-button type="primary" class="transfer" @click="sendRPG">
            sendRPG
          </el-button>
        </div>
        <div v-else>
          <el-button class="transfer" @click="sendToken"> sendToken </el-button>
          <br />
          <br />
          <el-button type="primary" class="transfer" @click="executeCall">
            executeCall
          </el-button>
        </div>

        <div>{{ txHash }}</div>
      </el-tab-pane>
      <el-tab-pane label="Sign Message" name="sign_message">
        <div>
          <br />
          <h3 class="input">Message:</h3>
          <el-input
            v-model="message"
            type="textarea"
            :autosize="{ minRows: 8, maxRows: 10 }"
            resize="none"
          ></el-input>
          <br />
          <div class="message">
            <el-button type="primary" class="message-button" @click="authorize"
              >authorize</el-button
            >
            <el-button type="primary" class="message-button" @click="verifySig"
              >verify</el-button
            >
          </div>
          <br />
          <div v-if="sig">
            <h3 class="input">Signature:</h3>
            <el-input
              v-model="sig"
              type="textarea"
              :autosize="{ minRows: 8, maxRows: 10 }"
              resize="none"
            ></el-input>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { UPAuthMessage, UPAuthResponse } from 'up-core-test'
import { ChainID, UPRangers } from 'up-rangers'
import { AbiItem, fromWei } from 'web3-utils'
import { ERC20ABI } from '~/assets/js/erc20.abi'

const DAI_ADDRESS = '0x25c58Aa062Efb4f069bD013De3e3C5797fb40651'

export default Vue.extend({
  data() {
    return {
      username: '',
      message: 'TO BE SIGNED MESSAGE abc',
      sig: '',
      activeTab: 'sign_transaction',
      tokenType: 'RPG',
      myAddress: '',
      myRPGBalance: '0.00',
      myTokenBalance: '0.00',
      toAddress: '0x8291507Afda0BBA820efB6DFA339f09C9465215C',
      toAmount: '0.01',
      toFeeAmount: '0.000001',
      toDescription: '描述测试描述测试描述测试',
      toTheme: 'dark',
      txHash: '',
      form: {},
      // STEP 1: create UPRangers instance
      upRangers: new UPRangers({
        chainID: ChainID.testnet,
        userInfoContract: process.env.RANGERS_UNIPASS_CONTRACT,
        upCoreConfig: {
          domain: process.env.UNIPASS_URL,
          // domain: 'localhost:3000',
          // protocol: 'http',
        },
      }),
      config: {
        entry: process.env.RANGERS_UNIPASS_CONTRACT,
      },
    }
  },
  computed: {
    myBalanceFormat(): string {
      const balance =
        this.tokenType === 'RPG' ? this.myRPGBalance : this.myTokenBalance

      return `${balance} ${this.tokenType}`
    },
  },
  methods: {
    bindCopy() {
      this.$clipboard(this.myAddress)
      this.$message.success('copy succeeded')
    },
    async connect() {
      console.log('connect clicked')
      try {
        // STEP 2: connect unipass
        const account = await this.upRangers.getUPCore().connect({
          email: true,
          evmKeys: true,
          chain: {
            id: 9527,
            name: 'Rangers Protocol Testnet Robin',
          },
          theme: this.toTheme as any,
        })
        this.username = account.username
        console.log('account', account)

        // STEP 3: init unipass with username and email
        await this.upRangers.initUniPass(this.username, account.email!)
        this.myAddress = this.upRangers.getAddress()

        await this.refreshBalance()
      } catch (err) {
        this.$message.error(err as string)
        console.log('connect err', err)
      }
    },
    async checkTxStatus(txHash: string) {
      let tryTimes = 0
      while (tryTimes++ < 3) {
        const receipt = await this.upRangers
          .getWeb3()
          .eth.getTransactionReceipt(txHash)

        if (receipt) return receipt.status
        await new Promise((resolve) => setTimeout(resolve, 1000))
      }
      return false
    },
    async onAddressChanged() {
      try {
        if (!this.upRangers.getWeb3().utils.isAddress(this.toAddress)) {
          this.toAddress = await this.upRangers.getAddressByEmail(
            this.toAddress,
          )
        }
      } catch (err) {
        this.$message.error(err as string)
        console.log('getAddress by email err', err)
      }
    },
    async refreshBalance() {
      this.myRPGBalance = fromWei(
        await this.upRangers.getWeb3().eth.getBalance(this.myAddress),
      )

      const tokenContract = new (this.upRangers.getWeb3().eth.Contract)(
        ERC20ABI as AbiItem[],
        DAI_ADDRESS,
      )
      this.myTokenBalance = fromWei(
        await tokenContract.methods.balanceOf(this.myAddress).call(),
      )
    },
    logout() {
      console.log('connect clicked')
      this.upRangers.getUPCore().disconnect()
      this.username = ''
    },
    async authorize() {
      console.log('authorize clicked')
      this.sig = ''
      console.log({
        username: this.username,
        message: this.message,
      })
      try {
        // SIGN Message with UniPass
        const resp = await this.upRangers
          .getUPCore()
          .authorize(
            new UPAuthMessage('PLAIN_MSG', this.username, this.message),
          )
        console.log('resp', resp)
        this.sig = JSON.stringify(resp)
      } catch (err) {
        this.$message.error(err as string)
        console.log('auth err', err)
      }
    },

    async verifySig() {
      try {
        // VERIFY user signed message and sig
        const ret = await this.upRangers.verifyUserSig(
          '0x' + Buffer.from(this.message, 'utf-8').toString('hex'),
          JSON.parse(this.sig) as UPAuthResponse,
        )
        if (ret === true) {
          this.$message.success('verify signature success')
        } else {
          this.$message.error('verify signature failed')
        }
      } catch (err) {
        this.$message.error(err as string)
        console.log('auth err', err)
      }
    },
    async sendRPG() {
      if (Number(this.myRPGBalance) < Number(this.toAmount)) {
        this.$message.error('balance is not enough')
        return
      }
      try {
        this.upRangers.getUPCore().initPop()

        // SEND RPG
        this.txHash = await this.upRangers.transferNativeToken(
          this.toAddress,
          this.upRangers.getWeb3().utils.toWei(this.toAmount),
          {
            feeToken: {
              address: '0x0000000000000000000000000000000000000000',
              symbol: 'RPG',
              decimals: 18,
            },
            feeAmount: this.upRangers.getWeb3().utils.toWei(this.toFeeAmount),
            description: this.toDescription,
          },
        )
        if (await this.checkTxStatus(this.txHash)) {
          console.log('send RPG success', this.txHash)
          this.$message.success(`send RPG success, tx hash = ${this.txHash}`)
        } else {
          this.$message.error(`send RPG failed, tx hash = ${this.txHash}`)
        }
        await this.refreshBalance()
      } catch (err) {
        this.$message.error(err as string)
        console.log('err', err)
      }
    },
    async sendToken() {
      if (Number(this.myTokenBalance) < Number(this.toAmount)) {
        this.$message.error('token balance is not enough')
        return
      }
      try {
        this.upRangers.getUPCore().initPop()

        // SEND DAI(ERC20) token
        this.txHash = await this.upRangers.transferToken(
          {
            address: DAI_ADDRESS,
            symbol: 'DAI',
            decimals: 18,
          },
          this.toAddress,
          this.upRangers.getWeb3().utils.toWei(this.toAmount),
          {
            feeToken: {
              address: '0x0000000000000000000000000000000000000000',
              symbol: 'RPG',
              decimals: 18,
            },
            feeAmount: this.upRangers.getWeb3().utils.toWei(this.toFeeAmount),
            description: this.toDescription,
          },
        )

        if (await this.checkTxStatus(this.txHash)) {
          console.log('send Token success', this.txHash)
          this.$message.success(`send token success, tx hash = ${this.txHash}`)
        } else {
          this.$message.error(`send token failed, tx hash = ${this.txHash}`)
        }
        await this.refreshBalance()
      } catch (err) {
        this.$message.error(err as string)
        console.log('err', err)
      }
    },
    async executeCall() {
      if (Number(this.myTokenBalance) < Number(this.toAmount)) {
        this.$message.error('token balance is not enough')
        return
      }
      try {
        this.upRangers.getUPCore().initPop()

        // CALL CONTRACT to execute its method
        this.txHash = await this.upRangers.executeCall(
          DAI_ADDRESS,
          '0x00',
          this.upRangers.getWeb3().eth.abi.encodeFunctionCall(
            {
              name: 'transfer',
              type: 'function',
              inputs: [
                {
                  name: 'dst',
                  type: 'address',
                },
                {
                  name: 'wad',
                  type: 'uint256',
                },
              ],
            },
            [
              this.toAddress,
              this.upRangers.getWeb3().utils.toWei(this.toAmount),
            ],
          ),
          {
            feeToken: {
              address: '0x0000000000000000000000000000000000000000',
              symbol: 'RPG',
              decimals: 18,
            },
            feeAmount: this.upRangers.getWeb3().utils.toWei(this.toFeeAmount),
            description: this.toDescription,
          },
        )
        await this.checkTxStatus(this.txHash)
        if (await this.checkTxStatus(this.txHash)) {
          console.log('execute call success', this.txHash)
          this.$message.success(
            `execute call success, tx hash = ${this.txHash}`,
          )
        } else {
          this.$message.error(`execute call failed, tx hash = ${this.txHash}`)
        }
        await this.refreshBalance()
      } catch (err) {
        this.$message.error(err as string)
        console.log('err', err)
      }
    },
  },
})
</script>

<style lang="stylus">
#page-demo {
  max-width: 480px;
  margin: 0 auto;
  overflow: hidden;
  position: relative;
  background: #F5F5FF;

  > * {
    z-index: 1;
  }

  > .background-logo {
    font-size: 237px;
    position: absolute;
    top: 16px;
    right: -40px;
    color: #5575ff;
    opacity: 0.14;
    z-index: 0;
  }

  .head {
    text-align: left;
    font-family: Helvetica;
    font-style: normal;
    font-weight: bold;
    font-size: 20px;
    line-height: 20px;
    color: black;
  }

  .transfer {
    width: 100%;
    font-size: 20px;
  }

  .login {
    margin-top: 50px;
    font-size: 20px;
  }

  .body {
    border-radius: 24px;
    margin: 30px auto 0px;
    width: 100%;
    background: #FFFFFF;
    padding: 0px 0 21px;
    overflow: hidden;

    .body-input {
      margin-top: -20px;

      .icon-copy {
        cursor: pointer;
      }
    }

    .input {
      text-align: left;
      margin-bottom: 20px;
    }
  }

  .message {
    display: flex;
    justify-content: space-between;

    .message-button {
      margin-top: 30px;
      width: 48%;
      font-size: 20px;
    }
  }
}

.unipass-page {
  padding: 24px;
  padding-top: 29px;
  width: 100%;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  text-align: center;
}
</style>
