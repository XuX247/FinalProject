<script setup>
import { ref, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useStore } from 'vuex'
import { Toast } from "../util/toast.js";

const route = useRoute()
const router = useRouter()
const store = useStore()

const pic = ref('')
const orderInfo = ref({})
const payType = ref('1')
const showQr = ref(false)

const hCoin = ref(0)
const finalPrice = ref(0)

onMounted(() => {
  const gameList = store.state.gameList
  const gameId = String(route.params.id)

  // 查找游戏
  for (let item of gameList) {
    if (String(item.id) === gameId) {
      orderInfo.value = item
      pic.value = item.covers[0]
      break
    }
  }

  // 计价游戏价钱夺少;冬促-20%
  const price = orderInfo.value.nowPrice || 0
  hCoin.value = Math.round(price * 0.2 * 100) / 100
  finalPrice.value = Math.round((price - hCoin.value) * 100) / 100
})

function handlePayClick() {
  // 安全检查：确保 orderInfo 有 id
  if (!orderInfo.value.id) {
    alert('商品信息加载失败')
    return
  }

  // 检查是否已拥有
  const ownGames = store.state.userInfo.ownGames || []
  for (let game of ownGames) {
    if (game.id === orderInfo.value.id) {
      alert('您已拥有该游戏！\n请勿重复购买！')
      return
    }
  }

  // 弹出二维码
  showQr.value = true
}

function confirmPayment() {
  // 添加到“我的游戏”
  store.commit('addGame', {
    id: orderInfo.value.id,
    totalTime: 0,
    recentTime: 0,
    myAchievements: 0,
    other: {}
  })

  showQr.value = false
  router.replace('/games')
  Toast('支付成功,去完游戏吧awa', 1000)

}
</script>

<template>
  <!-- 顶部 -->
  <div class="top">
    <div class="left" @click="router.go(-1)">←</div>
    <div class="center">确认订单</div>
    <div class="right"></div>
  </div>

  <div class="main">
    <!-- 商品信息 -->
    <div class="block">
      <div style="font-weight: bold; margin-bottom: 8px;">CDKEY（游戏）</div>
      <div class="game-item">
        <img :src="pic" class="cover" />
        <div class="info">
          <div style="font-size: 16px; font-weight: bold;">{{ orderInfo.name }}</div>
          <div style="font-size: 13px; color: #8f8f8f; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">
            数量×1；{{ orderInfo.name }}；CDKEY
          </div>
          <div style="font-size: 12px; color: #bbb;">CDKEY国区账号激活</div>
          <div style="font-size: 17px; font-weight: bold; color: black;">￥{{ orderInfo.nowPrice }}</div>
        </div>
      </div>
    </div>

    <!-- 支付方式 -->
    <div class="block">
      <div style="font-weight: bold; margin-bottom: 8px;">支付方式</div>
      <div class="pay-row" @click="payType = '1'">
        <span style="font-size: 20px; color: blue;">📱</span>
        <span style="margin-left: 10px; font-size: 16px;">支付宝支付</span>
        <span style="margin-left: auto;">{{ payType === '1' ? '●' : '' }}</span>
      </div>
      <div class="pay-row" @click="payType = '2'">
        <span style="font-size: 20px; color: #00af00;">💬</span>
        <span style="margin-left: 10px; font-size: 16px;">微信支付</span>
        <span style="margin-left: auto;">{{ payType === '2' ? '●' : '' }}</span>
      </div>
    </div>

    <!-- 价格明细 -->
    <div class="block">
      <div style="font-weight: bold; margin-bottom: 8px;">价格明细</div>
      <div class="price-line">
        <span>商品总价</span>
        <span>￥{{ orderInfo.nowPrice }}</span>
      </div>
      <div class="price-line">
        <span>H币抵现</span>
        <span>可抵￥{{ hCoin }}</span>
      </div>
      <div class="price-line" style="color: #8f8f8f;">
        <span>专属福利</span>
        <span>邀请Steam好友获得额外优惠 →</span>
      </div>
      <hr style="margin: 12px 0; border: none; border-top: 1px solid #eee;" />
      <div class="price-line" style="margin-top: 10px;">
        <span>实付款</span>
        <span style="color: black; font-size: 20px; font-weight: bold;">￥{{ finalPrice }}</span>
      </div>
    </div>

    <div style="text-align: center; color: #aaa; margin-top: 20px; font-size: 12px;">
      《大黑块商城购买协议》
    </div>
  </div>

  <!-- 底部按钮 -->
  <div class="bottom">
    <div>共1件</div>
    <div>合计: <span style="font-size: 20px; font-weight: bold;">￥{{ finalPrice }}</span></div>
    <div class="pay-btn" @click="handlePayClick">支付订单</div>
  </div>

  <!-- 付款弹窗 -->
  <div v-if="showQr" class="qr-mask" @click="showQr = false">
    <div class="qr-box" @click.stop>
      <img src="/ikun.png" width="250" height="250" style="margin-top: 20px;" />
      <div style="margin-top: 15px;">
        <div class="confirm-btn" @click="confirmPayment">已扫码付款</div>
      </div>
    </div>
  </div>
</template>



<style scoped>
.top {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 50px;
  background-color: white;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 10px;
  box-sizing: border-box;
  font-size: 16px;
  font-weight: bold;
  z-index: 100;
}

.main {
  padding: 60px 10px 80px;
  background-color: #f3f3f3;
}

.block {
  background-color: white;
  padding: 12px 15px;
  margin-bottom: 10px;
  border-radius: 4px;
}

.game-item {
  display: flex;
  align-items: center;
  margin-top: 10px;
}

.cover {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border: 1px solid #e9e9e9;
  border-radius: 3px;
  margin-right: 12px;
}

.pay-row {
  display: flex;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid #f5f5f5;
}

.pay-row:last-child {
  border-bottom: none;
}

.price-line {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 12px 0;
  color: #8d8d8d;
  font-weight: 550;
}

.bottom {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 60px;
  background-color: white;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 10px;
  box-sizing: border-box;
  font-size: 16px;
  color: #6e6e6e;
  box-shadow: 0 -1px 1px #f3f3f3;
}

.pay-btn {
  background-color: black;
  color: white;
  text-align: center;
  line-height: 45px;
  width: 140px;
  font-size: 18px;
  font-weight: bold;
  border-radius: 3px;
  cursor: pointer;
}

.qr-mask {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: flex-end;
  z-index: 1000;
}

.qr-box {
  background: white;
  width: 100%;
  padding: 20px;
  border-top-left-radius: 15px;
  border-top-right-radius: 15px;
  text-align: center;
}

.confirm-btn {
  background-color: black;
  color: white;
  padding: 10px 30px;
  font-weight: bold;
  border-radius: 4px;
  cursor: pointer;
  display: inline-block;
}
</style>