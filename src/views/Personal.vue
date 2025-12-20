<script setup>
import { computed, onMounted, ref } from 'vue'
import { useStore } from 'vuex'
import { useRouter } from 'vue-router'

const store = useStore()
const router = useRouter()

// 登录否？
const isLogin = computed(() => {
  return !!store.state.userInfo
})

// 获取用户信息，没登陆的话显示空拍头像
const user = computed(() => {
  if (isLogin.value) {
    return store.state.userInfo
  } else {
    return {
      name: '点击登录',
      signature: '登录解锁更多精彩内容~',
      avatar: '/head2.png',
      level: 0,
      atten: [],
      fans: [],
      praise: 0,
      browse: 0
    }
  }
})

// 游戏列表
const myGames = ref([])

// 按最近游玩时间降序排序
const sortedGames = computed(() => {
  return [...myGames.value].sort((a, b) => b.recentTime - a.recentTime) //"..."是复制数组用的,sort来排序
})

onMounted(() => {
  if (!isLogin.value) return;

  const userInfo = store.state.userInfo;
  const gameList = store.state.gameList;
  const ownGames = userInfo.ownGames || []; // 防止 undefined

  // 用来存最终结果：每款游戏都带上完整信息
  const gamesWithOther = [];

  // 遍历“我拥有的游戏”
  for (let i = 0; i < ownGames.length; i++) {
    const myGame = ownGames[i]; // 比如 { id: 101, totalTime: 5, ... }

    let fullGameInfo = null;

    // 在所有游戏中找匹配的 id
    for (let j = 0; j < gameList.length; j++) {
      if (gameList[j].id === myGame.id) {
        fullGameInfo = gameList[j]; // 找到了！
        break; // 找到就不用继续找了
      }
    }

    // 如果没找到，就用默认占位数据
    if (!fullGameInfo) {
      fullGameInfo = {
        name: '未知游戏',
        covers: ['/placeholder.png'],
        achievements: 0,
        avgTime: 10
      };
    }

    // 把“我的数据”和“游戏详情”拼在一起
    const mergedGame = {
      id: myGame.id,
      totalTime: myGame.totalTime,
      recentTime: myGame.recentTime,
      myAchievements: myGame.myAchievements,
      other: fullGameInfo
    };

    gamesWithOther.push(mergedGame);
  }

  // 赋值给响应式变量，页面就能显示了
  myGames.value = gamesWithOther;
});

// 跳转登录界面
function goLogin() {
  if (!isLogin.value) {
    router.push('/login')
  }
}

// 游戏详情
function goDetail(id) {
  router.push({ name: 'detail', params: { id } })
}

//百分比
function progressPercent(game) {
  if (!game.other.avgTime || game.other.avgTime <= 0) return 0
  const p = (game.totalTime / game.other.avgTime) * 100
  return Math.min(100, Math.round(p))
}

// 进度条颜色
function getProgressColor(game) {
  const p = progressPercent(game)
  if (p >= 90) return '#ff8e1e'
  if (p >= 70) return '#EE00BE'
  if (p >= 50) return '#2b80fd'
  return '#63f142'
}
</script>



<template>
  <!-- 顶部の导航-->
  <div class="top-nav">
    <div>数据</div>
    <div>动态</div>
  </div>

  <!-- 用户头部 -->
  <div class="head" @click="goLogin">
    <img :src="user.avatar" class="avatar" />
    <div class="user-info">
      <div>
        <span class="name">{{ user.name }}</span>
        <span v-if="isLogin" class="level">Lv.{{ user.level }}</span>
      </div>
      <span class="signature">{{ user.signature }}</span>
    </div>
  </div>


  <div v-if="isLogin" class="stats">
    <div class="stat-item">
      <div class="num">{{ user.atten.length }}</div>
      <div class="label">关注</div>
    </div>
    <div class="stat-item">
      <div class=" num">{{ user.fans.length }}</div>
      <div class="label">粉丝</div>
    </div>
    <div class="stat-item">
      <div class="num">{{ user.praise }}</div>
      <div class="label">获赞</div>
    </div>
    <div class="stat-item">
      <div class="num">{{ user.browse }}</div>
      <div class="label">历史浏览</div>
    </div>
  </div>

  <!-- 我的游戏 -->
  <div v-if="isLogin" class="games-section">
    <div class="tabs">
      <div class="tab">关注游戏</div>
      <div class="tab active">拥有游戏</div>
      <div class="tab">完美通关</div>
      <div class="tab">我的评分</div>
    </div>

    <div class="search-bar">
      <input type="text" placeholder="搜索" disabled />
      <span>总时长</span>
      <span>两周时长</span>
      <span>成就</span>
    </div>

    <div class="game-list">
      <div v-for="game in sortedGames" :key="game.id" class="game-item" @click="goDetail(game.id)">
        <img :src="game.other.covers[0]" class="game-cover" />
        <div class="game-main">
          <div class="game-name">{{ game.other.name }}</div>
          <div class="game-time">
            <span>{{ game.totalTime }}h</span>
            <span class="recent">{{ game.recentTime }}h</span>
          </div>
          <!-- 自制进度条 -->
          <div class="progress-bg">
            <div class="progress-fill"
              :style="{ width: progressPercent(game) + '%', backgroundColor: getProgressColor(game) }"></div>
          </div>
        </div>
        <div class="achievements">
          {{ game.myAchievements }} / {{ game.other.achievements }}
        </div>
      </div>
    </div>
  </div>
</template>



<style scoped>
.top-nav {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 40px;
  background: white;
  display: flex;
  justify-content: space-around;
  align-items: center;
  font-size: 14px;
  color: #7a7a7a;
  border-bottom: 1px solid #eee;
  z-index: 100;
}

.head {
  display: flex;
  align-items: center;
  padding: 10px;
  margin-top: 40px;
  cursor: pointer;
}

.avatar {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background: #e1e1e1;
  object-fit: cover;
}

.user-info {
  margin-left: 12px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.name {
  font-size: 20px;
  font-weight: bold;
}

.level {
  background: black;
  color: white;
  font-size: 12px;
  padding: 0 5px;
  border-radius: 3px;
  margin-left: 6px;
}

.signature {
  color: #777;
  letter-spacing: 2px;
  font-size: 14px;
}

.stats {
  display: flex;
  padding: 10px;
  background: white;
  margin: 10px;
  border-radius: 4px;
}

.stat-item {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
}

.num {
  font-size: 22px;
  font-weight: bold;
}

.label {
  color: #777;
  font-size: 14px;
}

.games-section {
  margin: 0 10px;
}

.tabs {
  display: flex;
  background: #f8f8f8;
  border-radius: 3px;
  height: 37px;
  align-items: center;
  font-size: 15px;
  color: #7a7a7a;
}

.tab {
  flex: 1;
  text-align: center;
  padding: 4px 0;
}

.tab.active {
  background: white;
  color: black;
  border-radius: 3px;
}

.search-bar {
  display: flex;
  align-items: center;
  padding: 8px 0;
  font-size: 14px;
  color: #777;
}

.search-bar input {
  width: 100px;
  padding: 4px;
  border: 1px solid #e0e0e0;
  border-radius: 3px;
  font-size: 14px;
}

.search-bar span {
  margin-left: 20px;
}

.game-list {
  margin-top: 10px;
}

.game-item {
  display: flex;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #e0e0e0;
  cursor: pointer;
}

.game-cover {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 4px;
  margin-right: 12px;
}

.game-main {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 80px;
}

.game-name {
  font-size: 16px;
  font-weight: bold;
}

.game-time {
  display: flex;
  justify-content: space-between;
  font-size: 14px;
  color: #333;
}

.recent {
  color: #aaa;
}

.progress-bg {
  width: 100%;
  height: 6px;
  background: #eee;
  border-radius: 3px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  transition: width 0.3s;
}

.achievements {
  font-size: 16px;
  font-weight: 550;
  min-width: 80px;
  text-align: right;
}
</style>