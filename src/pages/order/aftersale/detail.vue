<!-- 售后详情 -->
<template>
  <s-layout title="售后详情" :navbar="!isEmpty(state.info) && state.loading ? 'inner' : 'normal'">
    <view class="content_box" v-if="!isEmpty(state.info) && state.loading">
      <!-- 步骤条 -->
      <view
        class="steps-box ss-flex"
        :style="[
          {
            marginTop: '-' + Number(statusBarHeight + 88) + 'rpx',
            paddingTop: Number(statusBarHeight + 88) + 'rpx',
          },
        ]"
      >
        <!-- <uni-steps :options="state.list" :active="state.active" active-color="#fff" /> -->
        <view class="steps-items" :style="[{'grid-template-columns': `repeat(${state.list.length}, ${100/state.list.length}%)`}]">
          <view class="steps-item" v-for="(item, index) in state.list" :key="index">
            <view class="ss-flex">
              <text
                class="sicon-circlecheck"
                :class="state.active >= index ? 'activity-color' : 'info-color'"
              ></text>

              <view
                v-if="state.list.length - 1 != index"
                class="line"
                :class="state.active >= index ? 'activity-bg' : 'info-bg'"
              ></view>
            </view>
            <view
              class="steps-item-title"
              :class="state.active >= index ? 'activity-color' : 'info-color'"
              >{{ item.title }}</view
            >
          </view>
        </view>
      </view>

      <!-- 服务状态 -->
      <view
        class="status-box ss-flex ss-col-center ss-row-between ss-m-x-20"
      >
        <view class="">
          <view class="status-time">更新时间：{{ state.info.updateTime }}</view>
        </view>
      </view>

      <!-- 退款金额 -->
      <view class="aftersale-money ss-flex ss-col-center ss-row-between">
        <view class="aftersale-money--title">退款总额</view>
        <view class="aftersale-money--num">￥{{ state.info.returnAmount }}</view>
      </view>
      <!-- 服务商品 -->
      <view class="order-shop">
        <view class="border-bottom" v-for="item in state.info.orderItemList" :key="item.id">
          <s-goods-item
              :img="item.pic"
              :title="item.productName"
              :skuText="item.spDataValue"
              :price="item.salePrice"
              :num="item.quantity"
              priceColor="#333333"
              :titleWidth="480"
          >
          </s-goods-item>
        </view>
      </view>

      <!-- 填写退回单号 -->
      <view class="aftersale-code" v-if="state.info.type === 2 && state.info.status === 1 && !state.info.refundWaybillCode">
        <view class="refund-item u-m-b-20">
          <view class="item-title ss-m-b-20">
            <text>填写物流单号</text>
            <text class="import_tip">（退货地址请联系客服获取）</text>
          </view>
          <view class="flex_center item" @tap="state.showModal = true">
            <text class="flex-label">快递公司：</text>
            <view>
              <text class="ss-m-r-20 ss-p-l-20" v-if="state.returnOrder.deliveryCompanyCode">{{ state.returnOrder.companyName }}</text>
              <text class="ss-m-r-20 ss-p-l-20 grey" v-else>请选择快递公司</text>
            </view>
          </view>
          <view class="flex_center item">
            <text class="flex-label">快递单号：</text>
            <uni-easyinput
                :inputBorder="false"
                v-model="state.returnOrder.deliverySn"
                placeholder="请输入快递单号"
                paddingLeft="10"
            />
          </view>
          <view class="foot_box ss-flex ss-col-center ss-row-between">
            <button class="ss-reset-button btn ui-BG-Main-Gradient" @tap="submitReturnCode">提交</button>
          </view>
        </view>
      </view>
      <view class="aftersale-code" v-else-if="state.info.type === 2 && state.info.status === 1 && state.info.refundWaybillCode">
        <view class="item-title ss-m-b-20">
          <text>退货物流单号</text>
        </view>
        <view class="item ss-flex ss-col-center">
          <view class="item-title">快递公司：</view>
          <view class="item-content ss-m-r-16">{{companyMap[state.info.refundWpCode]}}</view>
        </view>
        <view class="item ss-flex ss-col-center">
          <view class="item-title">快递单号：</view>
          <view class="item-content ss-m-r-16">{{state.info.refundWaybillCode}}</view>
        </view>
<!--        <view class="foot_box ss-flex ss-col-center ss-row-between">-->
<!--          <button class="ss-reset-button btn" @tap="seeExpress" style="width: 220rpx">查看物流信息</button>-->
<!--        </view>-->
      </view>

      <!-- 服务内容 -->
      <view class="aftersale-content">
        <view class="aftersale-item ss-flex ss-col-center">
          <view class="item-title">服务单号：</view>
          <view class="item-content ss-m-r-16">{{ state.info.id }}</view>
          <button class="ss-reset-button copy-btn" @tap="onCopy">复制</button>
        </view>
        <view class="aftersale-item ss-flex ss-col-center">
          <view class="item-title">申请时间：</view>
          <view class="item-content">{{ state.info.createTime }}</view>
        </view>
        <view class="aftersale-item ss-flex ss-col-center">
          <view class="item-title">售后类型：</view>
          <view class="item-content">{{ state.info.type === 1 ? '仅退款':'退款退货' }}</view>
        </view>
        <view class="aftersale-item ss-flex ss-col-center">
          <view class="item-title">申请原因：</view>
          <view class="item-content">{{ state.info.reason }}</view>
        </view>
        <view class="aftersale-item ss-flex ss-col-center">
          <view class="item-title">相关描述：</view>
          <view class="item-content">{{ state.info.description }}</view>
        </view>
      </view>
    </view>
    <s-empty
      v-if="isEmpty(state.info) && state.loading"
      icon="/static/order-empty.png"
      text="暂无该订单售后详情"
    />
    <su-fixed bottom placeholder bg="bg-white" v-if="!isEmpty(state.info)">
      <view class="foot_box">
        <button
          class="ss-reset-button btn"
          v-if="state.info.status !== 2"
          @tap="onApply(state.info.orderId)"
          >取消售后</button
        >
        <!-- <button
          class="ss-reset-button btn"
          v-if="state.info.btns?.includes('delete')"
          @tap="onDelete(state.info.id)"
          >删除</button
        > -->
<!--        <button class="ss-reset-button contcat-btn btn" @tap="sheep.$router.go('/pages/chat/index')"-->
<!--          >联系客服</button-->
<!--        >-->
      </view>
    </su-fixed>

    <su-popup :show="state.showModal" round="10" :showClose="true" @close="state.showModal = false">
      <view class="modal-box page_box">
        <view class="modal-head item-title head_box ss-flex ss-row-center ss-col-center"
        >快递公司</view
        >
        <view class="modal-content content_box">
          <radio-group @change="onChange">
            <label
                class="radio ss-flex ss-col-center"
                v-for="item in companyList"
                :key="item.expressCode"
            >
              <view class="ss-flex-1 ss-p-20">{{ item.expressName }}</view>
              <radio
                  :value="item.expressCode"
                  color="var(--ui-BG-Main)"
                  :checked="item.expressCode === state.currentValue"
              />
            </label>
          </radio-group>
        </view>
        <view class="modal-foot foot_box ss-flex ss-row-center ss-col-center">
          <button class="ss-reset-button close-btn ui-BG-Main-Gradient" @tap="onReason"
          >确定</button
          >
        </view>
      </view>
    </su-popup>
  </s-layout>
</template>

<script setup>
  import sheep from '@/sheep';
  import { onLoad } from '@dcloudio/uni-app';
  import {computed, reactive} from 'vue';
  import { isEmpty } from 'lodash';

  const statusBarHeight = sheep.$platform.device.statusBarHeight * 2;
  const headerBg = sheep.$url.css('/icons/order_bg.png');
  const state = reactive({
    active: 0,
    orderId: null,
    aftersaleId: 0,
    info: {},
    currentValue: '',
    list: [
      {
        title: '提交申请',
      },
      {
        title: '处理中',
      },
    ],
    loading: false,
    returnOrder: {deliverySn: null,deliveryCompanyCode:null,companyName: ''}
  });

  const companyList = computed(()=>sheep.$store('user').companyList)
  const companyMap = computed(()=>sheep.$store('user').companyMap)

  function seeExpress(){
    sheep.$router.go('/pages/order/express/list', {
      waybillCode:state.info.refundWaybillCode,
    });
  }

  //选择申请原因
  function onChange(e) {
    state.currentValue = e.detail.value;
  }

  function submitReturnCode(){
    if (!state.returnOrder.deliveryCompanyCode || !state.returnOrder.deliverySn) {
      sheep.$helper.toast('请填写快递单号并选择快递公司');
      return;
    }
    const {deliverySn,deliveryCompanyCode} = state.returnOrder
    const params = {orderId:Number(state.orderId),deliveryCompanyCode,deliverySn}
    sheep.$api.order.aftersale.returnWaybillCode(params)
        .then(()=>{
          setTimeout(()=>{
            getDetail(state.orderId)
          },500)
        })
  }

  function onReason() {
    companyList.value.forEach((item) => {
      if (item.expressCode === state.currentValue) {
        state.returnOrder.companyName = item.expressName
      }
    });
    state.returnOrder.deliveryCompanyCode = state.currentValue;
    state.showModal = false;
  }

  function onApply(orderId) {
    uni.showModal({
      title: '提示',
      content: '确定要取消此申请吗？',
      success: async function (res) {
        if (res.confirm) {
          await sheep.$api.order.aftersale.cancel(orderId);
          sheep.$helper.toast('取消成功')
          sheep.$router.go('/pages/order/list',{type: 4});
        }
      },
    });
  }

  function onDelete(orderId) {
    uni.showModal({
      title: '提示',
      content: '确定要删除吗？',
      success: async function (res) {
        if (res.confirm) {
          const { error } = await sheep.$api.order.aftersale.delete(orderId);
          if (error === 0) {
            sheep.$router.back();
          }
        }
      },
    });
  }
  const onCopy = () => {
    sheep.$helper.copyText(state.info.id);
  };
  async function getDetail(id) {
    const data = await sheep.$api.order.aftersale.detail(id);
    state.loading = true;
    if (data) {
      data.orderItemList.forEach(item => {
        let str = ''
        const obj = JSON.parse(item.spData)
        Object.keys(obj).forEach(key => {
          str += key + "：" + obj[key] + " "
        })
        item.spDataValue = str
      })
      state.info = data;
      if (state.info.type === 1){
        //仅退款
        state.list.push({title: '已完成'})
        if (state.info.status === 2){
          state.active = 2
        }else {
          state.active = 1
        }
      }else {
        //退货退款
        state.list.push({ title: '退货中' });
        state.list.push({ title: '已完成' });
        if (state.info.status === 2) {
          state.active = 3;
        } else if (state.info.status === 1) {
          //退货中
          state.active = 2;
        } else if (state.info.status === 0) {
          state.active = 1;
        }
      }
    } else {
      state.info = null;
    }
  }
  onLoad((options) => {
    state.orderId = options.id;
    getDetail(options.id);
  });
</script>

<style lang="scss" scoped>

.modal-box {
  border-radius: 30rpx 30rpx 0 0;
  background: #fff;

  .modal-head {
    height: 100rpx;
    font-size: 30rpx;
  }

  .modal-content {
    font-size: 28rpx;
    padding: 0 20rpx;
    height: 800rpx;
    overflow-y: auto;
  }

  .modal-foot {
    justify-content: center !important;
    .close-btn {
      width: calc(100% - 40rpx);
      line-height: 80rpx;
      border-radius: 40rpx;
      color: rgba(#fff, 0.9);
    }
  }
}
// 步骤条
.steps-box {
  width: 100%;
  height: 190rpx;
  background: v-bind(headerBg) no-repeat,
  linear-gradient(90deg, var(--ui-BG-Main), var(--ui-BG-Main-gradient));
  background-size: 750rpx 100%;
  padding-left: 72rpx;
  .steps-items {
    display: grid;
    width: 100%;
  }
  .steps-item {
    .sicon-circleclose {
      font-size: 24rpx;
      color: #fff;
    }
    .sicon-circlecheck {
      font-size: 24rpx;
    }
    .steps-item-title {
      font-size: 24rpx;
      font-weight: 400;
      margin-top: 16rpx;
      margin-left: -36rpx;
      width: 100rpx;
      text-align: center;
    }
  }
}
.activity-color {
  color: #fff;
}
.info-color {
  color: rgba(#fff, 0.4);
}
.activity-bg {
  background: #fff;
}
.info-bg {
  background: rgba(#fff, 0.4);
}

.import_tip {
  color: red;
  //font-weight: bolder;
}
.flex_center {
  display: flex;
  align-items: center;
}
.flex-label {
  width: 150rpx;
}
.grey {
  color: #CCCCCC;
}

.line {
  width: 100%;
  height: 4rpx;
}

// 服务状态
.status-box {
  position: relative;
  z-index: 3;
  background-color: #fff;
  border-radius: 20rpx 20rpx 0px 0px;
  padding: 20rpx;
  margin-top: -20rpx;

  .status-text {
    font-size: 28rpx;

    font-weight: 500;
    color: rgba(51, 51, 51, 1);
    margin-bottom: 20rpx;
  }

  .status-time {
    font-size: 24rpx;

    font-weight: 400;
    color: rgba(153, 153, 153, 1);
  }
}
.aftersale-code {
  background-color: #fff;
  padding: 20rpx 20rpx 0 20rpx;
  margin: 20rpx;

  .item {
    height: 60rpx;
  }
}

// 退款金额
.aftersale-money {
  background-color: #fff;
  padding: 20rpx 20rpx;
  margin: 20rpx;

  .aftersale-money--title {
    font-size: 28rpx;

    font-weight: 500;
    color: rgba(51, 51, 51, 1);
  }

  .aftersale-money--num {
    font-size: 28rpx;
    font-family: OPPOSANS;
    font-weight: 500;
    color: #ff3000;
  }
}

// order-shop
.order-shop {
  padding: 20rpx;
  background-color: #fff;
  margin: 0 20rpx 20rpx 20rpx;
}

// 服务内容
.aftersale-content {
  background-color: #fff;
  padding: 20rpx;
  margin: 0 20rpx;
  margin-bottom: 20rpx;

  .aftersale-item {
    height: 60rpx;

    .copy-btn {
      background: #eeeeee;
      color: #333;
      border-radius: 20rpx;
      width: 75rpx;
      height: 40rpx;
      font-size: 22rpx;
    }

    .item-title {
      color: #999;
      font-size: 28rpx;
    }

    .item-content {
      color: #333;
      font-size: 28rpx;
    }
  }
}

// 底部功能
.foot_box {
  height: 100rpx;
  background-color: #fff;
  display: flex;
  align-items: center;
  justify-content: flex-end;

  .btn {
    width: 160rpx;
    height: 55rpx;
    border-radius: 30rpx;
    font-size: 26rpx;
    border: 2rpx solid #dcdcdc;
    line-height: normal;
    margin-right: 10rpx;
  }
}
</style>
