<template>
  <su-popup
    :show="show"
    type="bottom"
    round="20"
    @close="emits('close')"
    showClose
    backgroundColor="#f2f2f2"
  >
    <view class="model-box">
      <view class="title ss-m-t-16 ss-m-l-20 ss-flex">优惠券</view>
      <scroll-view
        class="model-content"
        scroll-y
        :scroll-with-animation="false"
        :enable-back-to-top="true"
      >
        <view class="subtitle ss-m-l-20">可使用优惠券</view>
        <view v-for="(item, index) in state.couponInfo" :key="index">
          <s-coupon-list :data="item" type="user" :disabled="false">
            <template #default>
              <checkbox-group @change="radioChange(item.id)" class="ss-flex ss-col-center">
                <label class="radio">
                  <checkbox
                    color="var(--ui-BG-Main)"
                    style="transform: scale(0.8)"
                    :checked="state.couponId === item.id"
                  />
                </label>
              </checkbox-group>
            </template>
          </s-coupon-list>
        </view>
<!--        <view class="subtitle ss-m-t-40 ss-m-l-20">不可使用优惠券</view>-->
<!--        <view v-for="item in state.couponInfo.cannot_use" :key="item.id">-->
<!--          <s-coupon-list :data="item" type="user" :disabled="true">-->
<!--            <template v-slot:reason>-->
<!--              <view class="ss-flex ss-m-t-24">-->
<!--                <view class="reason-title"> 不可用原因：</view>-->
<!--                <view class="reason-desc">{{ item.cannot_use_msg }}</view>-->
<!--              </view>-->
<!--            </template>-->
<!--          </s-coupon-list>-->
<!--        </view>-->
      </scroll-view>
    </view>
    <view class="modal-footer ss-flex">
      <button class="confirm-btn ss-reset-button" @tap="onConfirm">确认</button>
    </view>
  </su-popup>
</template>
<script setup>
  import { computed, reactive } from 'vue';
  const props = defineProps({
    modelValue: {
      type: Object,
      default() {},
    },
    show: {
      type: Boolean,
      default: false,
    },
  });
  const emits = defineEmits(['confirm', 'close']);
  const state = reactive({
    couponInfo: computed(() => props.modelValue),
    couponId: 0,
  });
  function radioChange(couponId) {
    if (state.couponId == couponId) {
      state.couponId = null;
    } else {
      state.couponId = couponId;
    }
  }
  const onConfirm = () => {
    emits('confirm', state.couponId);
  };
</script>
<style lang="scss" scoped>
  :deep() {
    checkbox .wx-checkbox-input {
      border-radius: 50% !important;
      color: #ffffff !important;
    }

    checkbox .wx-checkbox-input.wx-checkbox-input-checked {
      color: #fff;
      background: var(--ui-BG-Main);
    }

    .wx-checkbox-input.wx-checkbox-input-checked {
      border: none !important;
    }
  }

  .model-box {
    height: 60vh;
  }
  .title {
    font-size: 36rpx;
    height: 80rpx;
    font-weight: bold;
    color: #333333;
  }
  .subtitle {
    font-size: 26rpx;
    font-weight: 500;
    color: #333333;
  }
  .model-content {
    height: 54vh;
  }
  .modal-footer {
    width: 100%;
    height: 120rpx;
    background: #fff;
  }
  .confirm-btn {
    width: 710rpx;
    margin-left: 20rpx;
    height: 80rpx;
    background: linear-gradient(90deg, var(--ui-BG-Main), var(--ui-BG-Main-gradient));
    border-radius: 40rpx;
    color: #fff;
  }
  .reason-title {
    font-weight: 600;
    font-size: 20rpx;
    line-height: 26rpx;
    color: #ff0003;
  }
  .reason-desc {
    font-weight: 600;
    font-size: 20rpx;
    line-height: 26rpx;
    color: #434343;
  }
</style>
