<template>
    <div>
        <div class="wrapper">
            <textarea v-model="mnemonic" :class="{
                'center': !mnemonic
            }" :placeholder="$t('mnemonic.placeholder')"></textarea>
            <span v-show="errMsg" class="msg __err_msg" >
                {{ errMsg === 'mnemonic.empty' || errMsg === 'mnemonic.error' || errMsg === 'hint.nodeErr' ? $t(errMsg) : errMsg }}
            </span>
        </div>

        <create ref="createDom" :submit="validMnemonic"></create>

        <div class="note">{{ $t('mnemonic.hint') }}</div>

        <div class="__btn_list">
            <span class="__btn __btn_border __pointer" @click="leftClick" >{{ $t(leftTxt) }}</span>
            <div class="__btn __btn_all_in __pointer" @click="valid">
                <span v-show="!isLoading">{{ $t('startCreate.finish') }}</span>
                <loading v-show="isLoading" loadingType="dot"></loading>
            </div>
        </div>
    </div>
</template>

<script>
import create from './create.vue';
import loading from 'components/loading.vue';

export default {
    components: {
        create, loading
    },
    mounted() {
        this.$onKeyDown(13, () => {
            this.valid();
        });
    },
    props: {
        leftClick: {
            type: Function,
            default: () => {}
        },
        leftTxt: {
            type: String,
            default: ''
        },
        finishCb: {
            type: Function,
            default: () => {}
        }
    },
    data() {
        return {
            mnemonic: '',
            errMsg: '',
            isLoading: false
        };
    },
    methods: {
        valid() {
            this.$refs.createDom && this.$refs.createDom.valid();                   
        },
        validMnemonic(name, pass) {
            if (this.isLoading) {
                return;
            }

            let mnemonic = this.$trim(this.mnemonic);
            if (!mnemonic) {
                this.errMsg = 'mnemonic.empty';
                return;
            }

            this.isLoading = true;
            this.$wallet.restoreAddrs(mnemonic).then(()=>{
                this.restoreAccount(name, pass);
            }).catch(err => {
                console.warn(err);
                if (err && err.code === 500005) {
                    this.errMsg = 'mnemonic.error';
                } else {
                    this.errMsg = 'hint.nodeErr';
                }
                this.isLoading = false;
            });
        },
        restoreAccount(name, pass) {
            this.isLoading = true;
            this.$wallet.restoreAccount(name, pass).then(() => {
                if (!this.isLoading) {
                    return;
                }
                this.isLoading = false;

                let activeAccount = this.$wallet.getActiveAccount();
                activeAccount.rename(name);
                activeAccount.save();

                this.finishCb && this.finishCb();
                this.$router.push({
                    name: 'start'
                });
            }).catch((err) => {
                this.isLoading = false;
                console.warn(err);
                this.$toast(this.$t('hint.err'));
            });
        }
    }
};
</script>

<style lang="scss" scoped>
.wrapper {
    box-sizing: border-box;
    position: relative;
    background: #fff;
    border-radius: 3px;
    text-align: center;
    font-size: 14px;
    color: #1D2024;
    box-sizing: border-box;
    position: relative;
    padding: 20px;
    height: 100px;
    color: rgba(94,104,117,0.30);
    margin-bottom: 20px;
    textarea {
        width: 100%;
        height: 100%;
        resize: none;
        text-align: left;
        word-wrap: break-word;
        &.center {
            text-align: center;
            line-height: 60px;
        }
    }
    .msg {
        position: absolute;
        left: 0;
        bottom: 0;
    }
}
.note {
    font-size: 14px;
    color: #FFFFFF;
    text-align: left;
    line-height: 20px;
    margin: 30px 0;
}
</style>