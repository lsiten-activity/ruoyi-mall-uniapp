<template>
  <!-- 规格弹窗 -->
  <su-popup :show="authType !== ''" round="10" :showClose="true" @close="closeAuthModal" :type="type">
    <view class="login-wrap">
      <!-- 1. 账号密码登录 accountLogin -->
      <account-login
        v-if="authType === 'accountLogin'"
        :agreeStatus="state.protocol"
      ></account-login>

      <!-- 2.短信登录  smsLogin -->
      <sms-login v-if="authType === 'smsLogin'" :agreeStatus="state.protocol"></sms-login>
      <wechat-login v-if="authType === 'wechatMiniLogin'"></wechat-login>
      <!-- 3.短信注册 smsRegister-->
      <sms-register v-if="authType === 'smsRegister'" :agreeStatus="state.protocol"></sms-register>

      <!-- 4.忘记密码 resetPassword-->
      <reset-password v-if="authType === 'resetPassword'"></reset-password>

      <!-- 5.绑定手机号 changeMobile -->
      <change-mobile v-if="authType === 'changeMobile'"></change-mobile>

      <!-- 6.修改密码 changePassword-->
      <change-passwrod v-if="authType === 'changePassword'"></change-passwrod>

      <!-- 7.修改用户名 changeUsername-->
      <change-username v-if="authType === 'changeUsername'"></change-username>

<!--      联系客服-->
      <view v-if="authType === 'contact'">
        <mp-html class="contact-content" :content="contact"></mp-html>
      </view>

      <!-- 第三方登录 -->
      <view
        v-if="['accountLogin', 'smsLogin'].includes(authType)"
        class="auto-login-box ss-flex ss-row-center ss-col-center"
      >
        <!-- 微信小程序登录 -->
<!--        <button-->
<!--          v-if="sheep.$platform.name === 'WechatMiniProgram'"-->
<!--          open-type="getPhoneNumber"-->
<!--          @getphonenumber="thirdLogin('wechat', $event)"-->
<!--          class="ss-reset-button auto-login-btn"-->
<!--        >-->
<!--          <image-->
<!--            class="auto-login-img"-->
<!--            :src="sheep.$url.static('/static/img/shop/platform/wechat.png')"-->
<!--          ></image>-->
<!--        </button>-->

        <!-- 公众号|App微信登录 -->
<!--        <button-->
<!--          v-if="-->
<!--            ['WechatOfficialAccount', 'App'].includes(sheep.$platform.name) &&-->
<!--            sheep.$platform.isWechatInstalled-->
<!--          "-->
<!--          @tap="thirdLogin('wechat')"-->
<!--          class="ss-reset-button auto-login-btn"-->
<!--        >-->
<!--          <image-->
<!--            class="auto-login-img"-->
<!--            :src="sheep.$url.static('/static/img/shop/platform/wechat.png')"-->
<!--          ></image>-->
<!--        </button>-->

        <!-- iOS登录 -->
        <button
          v-if="sheep.$platform.os === 'ios' && sheep.$platform.name === 'App'"
          @tap="thirdLogin('apple')"
          class="ss-reset-button auto-login-btn"
        >
          <image
            class="auto-login-img"
            :src="sheep.$url.static('/static/img/shop/platform/apple.png')"
          ></image>
        </button>
      </view>

<!--      <view-->
<!--        v-if="['accountLogin', 'smsLogin', 'smsRegister'].includes(authType)"-->
<!--        class="agreement-box ss-flex ss-row-center"-->
<!--      >-->
<!--      <radio-group @change="onChange" class="ss-flex ss-col-center">-->
<!--          <label class="radio ss-flex">-->
<!--            <radio-->
<!--              :checked="state.protocol"-->
<!--              color="var(&#45;&#45;ui-BG-Main)"-->
<!--              style="transform: scale(0.8)"-->
<!--            />-->
<!--            <view class="agreement-text ss-flex ss-col-center ss-m-l-8">-->
<!--              我已阅读并遵守-->
<!--              <view-->
<!--                class="tcp-text"-->
<!--                @tap.stop="onProtocol(appInfo.user_protocol.id, appInfo.user_protocol.title)"-->
<!--              >-->
<!--                《{{ appInfo.user_protocol.title }}》-->
<!--              </view>-->
<!--              <view class="agreement-text">与</view>-->
<!--              <view-->
<!--                class="tcp-text"-->
<!--                @tap.stop="onProtocol(appInfo.privacy_protocol.id, appInfo.privacy_protocol.title)"-->
<!--              >-->
<!--                《{{ appInfo.privacy_protocol.title }}》-->
<!--              </view>-->
<!--            </view>-->
<!--          </label>-->
<!--        </radio-group>-->
<!--      </view>-->
      <view class="safe-box"></view>
    </view>
  </su-popup>
</template>

<script setup>
import {computed, onMounted, reactive, ref, watch} from 'vue';
  import sheep from '@/sheep';
  import accountLogin from './components/account-login.vue';
  import smsLogin from './components/sms-login.vue';
  import wechatLogin from './components/wechat-login.vue';
  import smsRegister from './components/sms-register.vue';
  import resetPassword from './components/reset-password.vue';
  import changeMobile from './components/change-mobile.vue';
  import changePasswrod from './components/change-password.vue';
  import changeUsername from './components/change-username.vue';
  import { closeAuthModal } from '@/sheep/hooks/useModal';

  const appInfo = computed(() => sheep.$store('app').info);

  const modalStore = sheep.$store('modal');

  const contact = ref({})
  // 授权弹窗类型
  const authType = computed(() => modalStore.auth);
  const type = computed(() => modalStore.type);

  const state = reactive({
    protocol: true,
  });

  //勾选协议
  function onChange() {
    state.protocol = !state.protocol;
  }

  // 查看协议
  function onProtocol(id, title) {
    closeAuthModal();
    sheep.$router.go('/pages/public/richtext', {
      id,
      title,
    });
  }

  // 第三方授权登陆
  const thirdLogin = async (provider, event = null) => {
    if (!state.protocol) {
      sheep.$helper.toast('请勾选同意');
      return;
    }
    const loginRes = await sheep.$platform.useProvider(provider).login(event?.detail || null);
    if (loginRes) {
      closeAuthModal();
      const userInfo = await sheep.$store('user').getInfo();
    }
  };

  const preAuthType = ref('')

  watch(
      () => authType.value,
      async () => {
        if (preAuthType.value === authType.value) {
          return;
        }
        //获取客服信息
        if(authType.value === 'contact') {
          const res = await sheep.$api.data.getSysConfig({configKey:'mall.contact'});
          contact.value = res.data || {configValue: ''}
        }
        preAuthType.value = 'contact'
      },
  );
</script>

<style lang="scss">
  @import './index.scss';

  .safe-box {
    height: calc(constant(safe-area-inset-bottom) / 5 * 3);
    height: calc(env(safe-area-inset-bottom) / 5 * 3);
  }

  .tcp-text {
    color: var(--ui-BG-Main);
  }

  .agreement-text {
    color: $dark-9;
  }

  .contact-content {
    text-align: center;
    image  {
      max-width: 640rpx !important; /* 设置最大宽度为容器的百分之百 */
    }
    rich-text {
      text-align: center;
    }
  }
</style>
