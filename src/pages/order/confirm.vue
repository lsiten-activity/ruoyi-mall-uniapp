<template>
  <s-layout title="确认订单">
    <view
      class="bg-white address-box ss-m-b-14 ss-r-b-10"
      @tap="onSelectAddress"
    >
      <s-address-item :item="state.addressInfo" :hasBorderBottom="false">
        <view class="ss-rest-button"><text class="_icon-forward"></text></view>
      </s-address-item>
    </view>
    <view class="order-card-box ss-m-b-14">
      <s-goods-item
        v-for="item in state.orderInfo.skuList"
        :key="item.skuId"
        :img="item.pic"
        :title="item.productName"
        :skuText="item.spDataValue"
        :price="item.price"
        :num="item.quantity"
        marginBottom="10"
      >
<!--        <template #top>-->
<!--          <view class="order-item ss-flex ss-col-center ss-row-between ss-p-x-20 bg-white">-->
<!--            <view class="item-title">配送方式</view>-->
<!--            <view class="ss-flex ss-col-center">-->
<!--              <text class="item-value">{{ item.dispatch_type_text }}</text>-->
<!--            </view>-->
<!--          </view>-->
<!--        </template>-->
      </s-goods-item>

      <view class="order-item ss-flex ss-col-center ss-row-between ss-p-x-20 bg-white ss-r-10">
        <view class="item-title">订单备注</view>
        <view class="ss-flex ss-col-center">
          <uni-easyinput
            maxlength="20"
            placeholder="填写订单备注信息"
            v-model="state.orderPayload.remark"
            :inputBorder="false"
            :clearable="false"
          ></uni-easyinput>
        </view>
      </view>
    </view>
    <!-- 合计 -->
    <view class="bg-white total-card-box ss-p-20 ss-m-b-14 ss-r-10">
      <view class="total-box-content border-bottom">
        <view class="order-item ss-flex ss-col-center ss-row-between">
          <view class="item-title">商品总金额</view>
          <view class="ss-flex ss-col-center">
            <text class="item-value ss-m-r-24">￥{{ state.orderInfo.productTotalAmount }}</text>
          </view>
        </view>
<!--        <view-->
<!--          class="order-item ss-flex ss-col-center ss-row-between"-->
<!--          v-if="state.orderPayload.order_type === 'score'"-->
<!--        >-->
<!--          <view class="item-title">扣除积分</view>-->
<!--          <view class="ss-flex ss-col-center">-->
<!--            <image-->
<!--              :src="sheep.$url.static('/static/img/shop/goods/score1.svg')"-->
<!--              class="score-img"-->
<!--            ></image>-->
<!--            <text class="item-value ss-m-r-24">{{ state.orderInfo.score_amount }}</text>-->
<!--          </view>-->
<!--        </view>-->
<!--        <view class="order-item ss-flex ss-col-center ss-row-between">-->
<!--          <view class="item-title">运费</view>-->
<!--          <view class="ss-flex ss-col-center">-->
<!--            <text class="item-value ss-m-r-24">+￥{{ state.orderInfo.dispatch_amount }}</text>-->
<!--          </view>-->
<!--        </view>-->
<!--        <view-->
<!--          class="order-item ss-flex ss-col-center ss-row-between"-->
<!--          v-if="state.orderPayload.order_type != 'score'"-->
<!--        >-->
        <view v-if="state.orderInfo.couponList?.length" class="order-item ss-flex ss-col-center ss-row-between">
          <view class="item-title">优惠券</view>
          <view class="ss-flex ss-col-center" @tap="state.showCoupon = true">
            <text class="item-value text-red" v-if="state.orderPayload.memberCouponId"
              >-￥{{ state.orderPayload.couponAmount }}</text
            >
            <text
              class="item-value text-red"
              v-else
              >{{
                state.orderInfo.couponList.length + '张可用'
              }}</text
            >
            <text class="_icon-forward item-icon"></text>
          </view>
        </view>
<!--        <view-->
<!--          class="order-item ss-flex ss-col-center ss-row-between"-->
<!--          v-if="state.orderInfo.promo_infos?.length"-->
<!--        >-->
<!--          &lt;!&ndash; <view v-if="state.orderInfo.promo_discount_fee > 0" class="order-item ss-flex ss-col-center ss-row-between"> &ndash;&gt;-->
<!--          <view class="item-title">活动优惠</view>-->
<!--          <view class="ss-flex ss-col-center" @tap="state.showDiscount = true">-->
<!--            <text class="item-value text-red"> -￥{{ state.orderInfo.promo_discount_fee }} </text>-->
<!--            <text class="_icon-forward item-icon"></text>-->
<!--          </view>-->
<!--        </view>-->
      </view>
      <view class="total-box-footer ss-font-28 ss-flex ss-row-right ss-col-center ss-m-r-28">
        <view class="total-num ss-m-r-20">共{{ state.totalNumber }}件</view>
        <view>合计：</view>
        <view class="total-num text-red"> ￥{{ state.orderPayload.payAmount }} </view>
        <view class="ss-flex" v-if="state.orderPayload.order_type === 'score'">
          <view class="total-num ss-font-30 text-red ss-m-l-4"> + </view>
          <image
            :src="sheep.$url.static('/static/img/shop/goods/score1.svg')"
            class="score-img"
          ></image>
          <view class="total-num ss-font-30 text-red">{{ state.orderPayload.payAmount }}</view>
        </view>
      </view>
    </view>
    <!-- 发票 -->
<!--    <view class="bg-white ss-p-20 ss-r-20">-->
<!--      <view class="order-item ss-flex ss-col-center ss-row-between">-->
<!--        <view class="item-title">发票申请</view>-->
<!--        <view class="ss-flex ss-col-center" @tap="onSelectInvoice">-->
<!--          <text class="item-value">{{ state.invoiceInfo.name || '无需开具发票' }}</text>-->
<!--          <text class="_icon-forward item-icon"></text>-->
<!--        </view>-->
<!--      </view>-->
<!--    </view>-->
    <!-- 选择优惠券弹框 -->
    <s-coupon-select
      v-model="state.orderInfo.couponList"
      :show="state.showCoupon"
      @confirm="onSelectCoupon"
      @close="state.showCoupon = false"
    />
    <!-- 满额折扣弹框  -->
<!--    <s-discount-list-->
<!--      v-model="state.orderInfo"-->
<!--      :show="state.showDiscount"-->
<!--      @close="state.showDiscount = false"-->
<!--    />-->
    <!-- 底部 -->
    <su-fixed bottom :opacity="false" bg="bg-white" placeholder :noFixed="false" :index="200">
      <view class="footer-box border-top ss-flex ss-row-between ss-p-x-20 ss-col-center">
        <view class="total-box-footer ss-flex ss-col-center">
          <view class="total-num ss-font-30 text-red"> ￥{{ state.orderPayload.payAmount }} </view>
        </view>

        <button
          class="ss-reset-button ui-BG-Main-Gradient ss-r-40 submit-btn ui-Shadow-Main"
          @tap="onConfirm"
        >
          {{ exchangeNow ? '立即兑换' : '提交订单' }}
        </button>
      </view>
    </su-fixed>
  </s-layout>
</template>

<script setup>
  import { reactive, computed } from 'vue';
  import { onLoad, onPageScroll, onShow } from '@dcloudio/uni-app';
  import sheep from '@/sheep';
  import { isEmpty } from 'lodash';
  import {numSub} from "@/sheep/hooks/useGoods";

  const state = reactive({
    orderPayload: {},
    orderInfo: {},
    addressInfo: {},
    invoiceInfo: {},
    totalNumber: 0,
    showCoupon: false,
    couponInfo: [],
    showDiscount: false,
  });

  // 立即兑换(立即兑换无需跳转收银台)
  const exchangeNow = computed(
    () => state.orderPayload.order_type === 'score' && state.orderInfo.pay_fee == 0,
  );

  // 选择地址
  function onSelectAddress() {
    uni.$once('SELECT_ADDRESS', (e) => {
      changeConsignee(e.addressInfo);
    });
    sheep.$router.go('/pages/user/address/list');
  }

  // 更改收货人地址&计算订单信息
  async function changeConsignee(addressInfo = {}) {
    if (isEmpty(addressInfo)) {
      const res = await sheep.$api.user.address.default();
      if (!isEmpty(res)) {
        addressInfo = res;
      }
    }
    if (!isEmpty(addressInfo)) {
      state.addressInfo = addressInfo;
      state.orderPayload.address_id = state.addressInfo.id;
    }
    getOrderInfo();
  }

  // 选择优惠券
  async function onSelectCoupon(e) {
    state.orderPayload.memberCouponId = e || null;
    //计算金额
    if (!state.orderPayload.memberCouponId) {
      state.orderPayload.couponAmount = 0
      state.orderPayload.payAmount = state.orderInfo.orderTotalAmount
    } else {
      const coupon = state.orderInfo.couponList.filter(it=>it.id == state.orderPayload.memberCouponId)[0]
      state.orderPayload.couponAmount = coupon.couponAmount
      let amount = numSub(state.orderInfo.orderTotalAmount,coupon.couponAmount);
      state.orderPayload.payAmount = amount > 0 ? amount : 0;
    }
    state.showCoupon = false;
  }

  // 选择发票信息
  function onSelectInvoice() {
    uni.$once('SELECT_INVOICE', (e) => {
      state.invoiceInfo = e.invoiceInfo;
      state.orderPayload.invoice_id = e.invoiceInfo.id || 0;
    });
    sheep.$router.go('/pages/user/invoice/list');
  }

  // 提交订单/立即兑换
  function onConfirm() {
    if (!state.orderPayload.address_id && state.orderInfo.need_address === 1) {
      sheep.$helper.toast('请选择收货地址');
      return;
    }
    submitOrder();
    // if (exchangeNow.value) {
    //   uni.showModal({
    //     title: '提示',
    //     content: '确定使用积分立即兑换?',
    //     cancelText: '再想想',
    //     success: async function (res) {
    //       if (res.confirm) {
    //         submitOrder();
    //       }
    //     },
    //   });
    // } else {
    // }
  }

  // 创建订单&跳转
  async function submitOrder() {
    const params = {
      addressId:state.orderPayload.address_id,
      note: state.orderPayload.remark,
      from: state.orderPayload.from,
      payType: 2,
      skuList: state.orderInfo.skuList,
      memberCouponId: state.orderPayload.memberCouponId
    }
    console.log('订单params：', params)
    const res = await sheep.$api.order.create(params);
      // sheep.$helper.toast('下单成功')
      // 更新购物车列表
      if (state.orderPayload.from === 'cart') {
        sheep.$store('cart').getList();
      }
      if (state.orderPayload.payAmount == 0) {
        sheep.$router.redirect('/pages/pay/result', {
          orderSN: res,
        });
      }
      sheep.$router.redirect('/pages/pay/index', {
        orderSN: res,
        totalAmount: state.orderPayload.payAmount,
        orderType: 'memberConsumer',
      });
      // if (exchangeNow.value) {
      //   sheep.$router.redirect('/pages/pay/result', {
      //     orderSN: data.order_sn,
      //   });
      // } else {
      //   sheep.$router.redirect('/pages/pay/index', {
      //     orderSN: data.order_sn,
      //   });
      // }
  }

  // 检查库存&计算订单价格
  async function getOrderInfo() {
    const skuList = state.orderPayload.goods_list.map(it => {return {skuId: it.skuId, quantity:it.goods_num}})
    let res = await sheep.$api.order.calc({skuList});
    if (!res) {
      setTimeout( () => {
        sheep.$router.back()
      }, 2000)
      return
    }
    state.totalNumber = 0
    state.orderInfo = res
    state.orderInfo.skuList.forEach( (item) => {
      let str = ''
      const obj = JSON.parse(item.spData)
      Object.keys(obj).forEach((key) => {
        str += key + ": " + obj[key] + "  "
      })
      item.spDataValue = str
      state.totalNumber += item.quantity
    })
    state.orderPayload.payAmount = state.orderInfo.orderTotalAmount
  }

  // 获取可用优惠券
  async function getCoupons() {
    const { error, data } = await sheep.$api.order.coupons(state.orderPayload);
    if (error === 0) {
      state.couponInfo = data;
    }
  }

  onLoad(async (options) => {
    if (options.data) {
      state.orderPayload = JSON.parse(options.data);
      changeConsignee();
      // if (state.orderPayload.order_type !== 'score') {
      //   getCoupons();
      // }
    }
  });
</script>

<style lang="scss" scoped>
  :deep() {
    .uni-input-wrapper {
      width: 320rpx;
    }

    .uni-easyinput__content-input {
      font-size: 28rpx;
      height: 72rpx;
      text-align: right !important;
      padding-right: 0 !important;

      .uni-input-input {
        font-weight: 500;
        color: #333333;
        font-size: 26rpx;
        height: 32rpx;
        margin-top: 4rpx;
      }
    }
    .uni-easyinput__content {
      display: flex !important;
      align-items: center !important;
      justify-content: right !important;
    }
  }
  .score-img {
    width: 36rpx;
    height: 36rpx;
    margin: 0 4rpx;
  }
  .order-item {
    height: 80rpx;

    .item-title {
      font-size: 28rpx;
      font-weight: 400;
    }

    .item-value {
      font-size: 28rpx;
      font-weight: 500;
    }

    .item-value {
      font-family: OPPOSANS;
    }
    .text-disabled {
      color: #bbbbbb;
    }

    .item-icon {
      color: $dark-9;
    }

    .remark-input {
      text-align: right;
    }

    .item-placeholder {
      color: $dark-9;
      font-size: 26rpx;
      text-align: right;
    }
  }

  .total-box-footer {
    height: 90rpx;

    .total-num {
      color: #333333;
      font-family: OPPOSANS;
    }
  }

  .footer-box {
    height: 100rpx;

    .submit-btn {
      width: 240rpx;
      height: 70rpx;
      font-size: 28rpx;
      font-weight: 500;

      .goto-pay-text {
        line-height: 28rpx;
      }
    }

    .cancel-btn {
      width: 240rpx;
      height: 80rpx;
      font-size: 26rpx;
      background-color: #e5e5e5;
      color: $dark-9;
    }
  }
  .title {
    font-size: 36rpx;
    font-weight: bold;
    color: #333333;
  }
  .subtitle {
    font-size: 28rpx;
    color: #999999;
  }
  .cicon-checkbox {
    font-size: 36rpx;
    color: var(--ui-BG-Main);
  }
  .cicon-box {
    font-size: 36rpx;
    color: #999999;
  }
</style>
