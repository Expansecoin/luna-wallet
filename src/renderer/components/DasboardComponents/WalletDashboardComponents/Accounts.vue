<template>
    <div class="userInfo">
        <div class="curve"></div>
        <div class="left">
            <!-- <div class="qrCode" id="qrCode1"> -->
            <div class="qrCodevue" >
                <qrcode-vue :value="accounthash" level="L"></qrcode-vue>
            </div>
            <!-- </div> -->
            <div class="info main-account">
                <h1 class="m_name">{{accountTitle? accountTitle: 'Main Account'}}</h1>
                <label class="balance">
                    {{defaultCurrencyData === '$'? '$':""}} {{ (parseFloat(accPriceData) * accountbalance).toFixed(5)}} {{defaultCurrencyData !== '$'? defaultCurrencyData:""}}
                </label>
                <!-- <span class="currency-sign">Exp</span> -->
                <div class=" copy accoundID wd300">
                    <label class="main-account-hash">{{accounthash? accounthash: '0X'}}</label>
                    <div @click="handletooltip" class="tooltip3 copytext" data-val=".main-account-hash-input">
                        <img src="../../../assets/img/copy.svg"  />
                        <span v-if="copiedtip" class="tooltiptext2">Copied</span>
                    </div>
                    <input type="hidden" v-model="accounthash" class="main-account-hash-input" />

                </div>
                <div class="btns">
                    <button class="button action button--shikoba md-trigger openAddAccountModal" @click="show" data-modal="modal-3">
                        <img class="button__icon" src="../../../assets/img/addaccount.svg" />
                        <span>Add Account</span>
                    </button>
                    <button class="button primery button--shikoba account1 a1 main_view_account" data-val="0xd416db48cce26ad2dff060d90ef45db8a67aa3ac"
                            @click="mainMenu()">
                        <img class="button__icon" src="../../../assets/img/viewaccount.svg" />
                        <span>View Account</span>
                    </button>
                </div>
            </div>
        </div>
        <modal class="modal" name="openAddAccountModal">
            <addAccount-app/>
        </modal>
    </div>
</template>

<script>
    import QrcodeVue from 'qrcode.vue';
    import {web3} from '../../../../main/libs/config';
    import { clipboard, dialog } from 'electron';
    // var web3 = startConnectWeb();
    import AddAccount from './AddAccount';
    export default {
        name: 'Accounts',
        components:{
            'addAccount-app': AddAccount,
            'qrcode-vue': QrcodeVue,
        },
        data() {
          return {
             accountTitle: '',
             accounthash: '',
             accountbalance: '',
              copiedtip: false,
          };
        },
        computed: {
            accounts() {
                this.expaccounts = this.$store.state.allAccounts;
                return this.expaccounts;
            },
            defaultCurrencyData() {
                this.defaultSign = this.$store.state.ac_dcurrency ;
                console.log(this.defaultSign, "this.defaultSign -----------------------------")
                return this.defaultSign;
            },
            accPriceData() {
                var curr = this.defaultCurrencyData === "$" ?  'USD':this.defaultCurrencyData;
                this.accbprice = this.$store.state.currencies && this.$store.state.currencies[curr].PRICE.replace(/[^0-9\.]/g, '');
                return this.accbprice;
            },
        },
        created(){
            console.log(web3,"web3")
            let that = this;
            that.intervalid1 = setInterval(() => {
                if (that.accounts.length > 0 && that.accounts !== ' ') {
                    that.accounts.map((account_hash) => {
                        if(account_hash){
                            web3.eth.getCoinbase().then((res)=> {
                                // console.log(res);
                                if(account_hash.hash) {
                                    if (res == (account_hash.hash).toLowerCase()){
                                        // console.log(account_hash, "account_hash");
                                        that.accountTitle = account_hash.accountTitle;
                                        that.accounthash = account_hash.hash;
                                        that.accountbalance = account_hash.balance;
                                        // console.log(that.accountbalance, "accountbalance");
                                        clearInterval(that.intervalid1)
                                    }
                                }
                            });
                        }
                    })
                }
            }, 100);
        },
        methods: {
            mainMenu(){
                this.$router.push({
                    path: '/accountdetails',
                    query: { accountDetail: this.accounthash }
                });
            },
            show () {
                this.$modal.show('openAddAccountModal');
            },
            hide () {
                this.$modal.hide('openAddAccountModal');
            },
            handletooltip(){
                this.copiedtip = true;
                var copyText = this.accounthash;
                if (copyText) {
                    clipboard.writeText(copyText, 'selected');
                }
                setTimeout(() => {
                    this.copiedtip = false;
                },2000);

            },
        }
    }
</script>

<style>
    .qrCodevue {
        padding: 10px!important;
        width: 101px!important;
        height: 101px!important;
        background: white!important;
    }
    .multiselect__select{
        top: 5px !important;
    }

</style>


<style>
    @import "../../../../../static/modalcomponent.css";
</style>
