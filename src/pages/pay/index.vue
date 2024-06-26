<!-- 收银台 -->
<template>
  <s-layout title="收银台">
    <view class="bg-white ss-modal-box ss-flex-col">
      <view class="modal-header ss-flex-col ss-col-center ss-row-center">
        <view class="money-box ss-m-b-20">
          <text class="money-text">{{ state.totalAmount }}</text>
        </view>
        <view class="time-text">
          <text>{{ payDescText }}</text>
        </view>
      </view>
      <view class="modal-content ss-flex-1">
        <view class="pay-title ss-p-l-30 ss-m-y-30">选择支付方式</view>
        <radio-group @change="onTapPay">
          <label class="pay-type-item" v-for="item in state.payMethods" :key="item.title">
            <view
              class="pay-item ss-flex ss-col-center ss-row-between ss-p-x-30 border-bottom"
              :class="{ 'disabled-pay-item': item.disabled }"
              v-if="allowedPayment.includes(item.value)"
            >
              <view class="ss-flex ss-col-center">
                <image
                  class="pay-icon"
                  v-if="item.disabled"
                  :src="sheep.$url.cdn('/static/img/shop/pay/cod_disabled.png')"
                  mode="aspectFit"
                ></image>
                <image
                  class="pay-icon"
                  v-else
                  :src="sheep.$url.cdn(item.icon)"
                  mode="aspectFit"
                ></image>
                <text class="pay-title">{{ item.title }}</text>
              </view>
              <view class="check-box ss-flex ss-col-center ss-p-l-10">
                <view class="userInfo-money ss-m-r-10" v-if="item.value == 'money'">
                  余额: {{ userInfo.money }}元
                </view>
                <view
                  class="userInfo-money ss-m-r-10"
                  v-if="item.value == 'offline' && item.disabled"
                >
                  部分商品不支持
                </view>
                <radio
                  :value="item.value"
                  color="var(--ui-BG-Main)"
                  style="transform: scale(0.8)"
                  :disabled="item.disabled"
                  :checked="state.payment === item.value"
                />
              </view>
            </view>
          </label>
        </radio-group>
      </view>
      <!-- 工具 -->
      <view class="modal-footer ss-flex ss-row-center ss-col-center ss-m-t-80 ss-m-b-40">
        <button v-if="state.payStatus === 0" class="ss-reset-button past-due-btn">
          检测支付环境中
        </button>
        <button v-else-if="state.payStatus === -1" class="ss-reset-button past-due-btn" disabled>
          支付已过期
        </button>
        <button
          v-else
          class="ss-reset-button save-btn"
          @tap="onPay"
          :disabled="state.payStatus !== 1"
          :class="{ 'disabled-btn': state.payStatus !== 1 }"
        >
          立即支付
        </button>
      </view>
    </view>
  </s-layout>
</template>
<script setup>
  import { computed, reactive } from 'vue';
  import { onLoad } from '@dcloudio/uni-app';
  import sheep from '@/sheep';
  import {Base64} from "js-base64";

  const userInfo = computed(() => sheep.$store('user').userInfo);

  // 检测支付环境
  const state = reactive({
    orderType: 'goods',
    payment: 'wechat',
    totalAmount: '',
    orderInfo: {},
    payStatus: 0, // 0=检测支付环境, -2=未查询到支付单信息， -1=支付已过期， 1=待支付，2=订单已支付
    payMethods: [],
    payId: ''
  });

  const allowedPayment = computed(() => {
    if(state.orderType === 'recharge') {
      return sheep.$store('app').platform.recharge_payment
    }
    return sheep.$store('app').platform.payment
    });

  const payMethods = [
    {
      icon: '/icons/wechat.png',
      title: '微信支付',
      value: 'wechat',
      disabled: false,
    },
    {
      icon: '/icons/alipay.png',
      title: '支付宝支付',
      value: 'alipay',
      disabled: false,
    },
    {
      icon: '/icons/wallet.png',
      title: '余额支付',
      value: 'money',
      disabled: false,
    },
    {
      icon: '/icons/apple.png',
      title: 'Apple Pay',
      value: 'apple',
      disabled: false,
    },
    {
      icon: '/icons/cod.png',
      title: '货到付款',
      value: 'offline',
      disabled: false,
    },
  ];

  const onPay = () => {
    if (sheep.$platform.name === 'H5') {
      sheep.$helper.toast('请在微信中打开支付')
      return;
    }
    if (state.payment === '') {
      sheep.$helper.toast('请选择支付方式');
      return;
    }
    if (state.payment === 'wechat') {
      uni.showModal({
        title: '提示',
        content: '确定要支付吗?',
        success: function (res) {
          if (res.confirm) {
            sheep.$platform.pay(state.payment, state.orderType, state.payId, state.totalAmount);
          }
        }
      })
    }
  }

  const payDescText = computed(() => {
    // if (state.payStatus === 2) {
    //   return '该订单已支付';
    // }
    // if (state.payStatus === 1 && state.orderInfo.ext.expired_time !== 0) {
    //   const time = useDurationTime(state.orderInfo.ext.expired_time);
    //   if (time.ms <= 0) {
    //     state.payStatus = -1;
    //     return '';
    //   }
    //   return `剩余支付时间 ${time.h}:${time.m}:${time.s} `;
    // }
    // if (state.payStatus === -2) {
    //   return '未查询到支付单信息';
    // }

    return '';
  });

  function checkPayStatus() {
    if (state.orderInfo.status === 'unpaid') {
      state.payStatus = 1;
      return;
    }
    if (state.orderInfo.status === 'closed') {
      state.payStatus = -1;
      return;
    }
    state.payStatus = 2;
  }

  function onTapPay(e) {
    state.payment = e.detail.value;
  }

  async function setRechargeOrder(id) {
    const { data, error } = await sheep.$api.trade.order(id);
    if (error === 0) {
      state.orderInfo = data;
      state.payMethods = payMethods;
      checkPayStatus();
    } else {
      state.payStatus = -2;
    }
  }

  async function setGoodsOrder(id) {
    state.payStatus = 1
    state.payMethods = payMethods.filter(it=>it.value==='wechat')
  }

  onLoad(async (options) => {
    const authInfo = sheep.$store('app').authInfo
    if (!sheep.$store('user').userInfo.openId && authInfo) {
      await sheep.$api.user.setWechatInfo(Base64.encode(JSON.stringify(authInfo)))
      sheep.$store('user').userInfo.openId = authInfo.openid
      // sheep.$store('app').authInfo = null
    }
    // if (
    //   sheep.$platform.name === 'WechatOfficialAccount' &&
    //   sheep.$platform.os === 'ios' &&
    //   !sheep.$platform.landingPage.includes('pages/pay/index')
    // ) {
    //   location.reload();
    //   return;
    // }
    let id = '';
    if (options.orderSN) {
      id = options.orderSN;
    }
    if (options.id) {
      id = options.id;
    }
    if (options.totalAmount){
      state.totalAmount = options.totalAmount
    }
    state.payId = id
    if (options.type === 'recharge') {
      state.orderType = 'recharge';
      // 充值订单
      setRechargeOrder(id);
    } else {
      state.orderType = 'goods';
      // 商品订单
      setGoodsOrder(id);
    }
  })
</script>

<style lang="scss" scoped>
  .pay-icon {
    width: 36rpx;
    height: 36rpx;
    margin-right: 26rpx;
  }

  .ss-modal-box {
    // max-height: 1000rpx;

    .modal-header {
      position: relative;
      padding: 60rpx 20rpx 40rpx;


      .money-text {
        color: $red;
        font-size: 46rpx;
        font-weight: bold;
        font-family: OPPOSANS;

        &::before {
          content: '￥';
          font-size: 30rpx;
        }
      }

      .time-text {
        font-size: 26rpx;
        color: $gray-b;
      }

      .close-icon {
        position: absolute;
        top: 10rpx;
        right: 20rpx;
        font-size: 46rpx;
        opacity: 0.2;
      }
    }

    .modal-content {
      overflow-y: auto;

      .pay-title {
        font-size: 26rpx;
        font-weight: 500;
        color: #333333;
      }

      .pay-tip {
        font-size: 26rpx;
        color: #bbbbbb;
      }

      .pay-item {
        height: 86rpx;
      }
      .disabled-pay-item {
        .pay-title {
          color: #999999;
        }
      }

      .userInfo-money {
        font-size: 26rpx;
        color: #bbbbbb;
        line-height: normal;
      }
    }

    .save-btn {
      width: 710rpx;
      height: 80rpx;
      border-radius: 40rpx;
      background: linear-gradient(90deg, var(--ui-BG-Main), var(--ui-BG-Main-gradient));
      color: $white;
    }
    .disabled-btn {
      background: #e5e5e5;
      color: #999999;
    }

    .past-due-btn {
      width: 710rpx;
      height: 80rpx;
      border-radius: 40rpx;
      background-color: #999;
      color: #fff;
    }
  }
</style>
