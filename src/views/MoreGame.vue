<script setup>
import { useRouter } from 'vue-router';
import { useStore } from 'vuex';

const router = useRouter();
const store = useStore();
let title = store.state.moreGame.title;
let games = store.state.moreGame.games;

function goBack() {
  router.go(-1);
}

function goToDetail(id) {
  router.push({ name: 'detail', params: { id: id } });
}
</script>





<template>
  <div class="top-bar">
    <div class="back-btn" @click="goBack">
      ←
    </div>
    <div class="title-text">
      {{ title }}
    </div>
    <div style="width: 24px"></div>
  </div>

  <div v-for="(game, index) in games" :key="index" class="game-item" @click="goToDetail(game.id)">
    <img :src="game.covers[0]" class="game-img" />
    <div class="game-info">
      <div class="game-name">{{ game.name }}</div>
      <div class="price-box">
        <img src="/xhh.png" class="coin-icon" />
        <span v-if="game.nowPrice !== 0">
          ￥{{ game.nowPrice }}
          <span v-if="game.nowPrice !== game.originalPrice" class="old-price">
            ￥{{ game.originalPrice }}
          </span>
        </span>
        <span v-else>免费</span>
      </div>
    </div>
  </div>
</template>






<style scoped>
/* 顶栏 */
.top-bar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 50px;
  background-color: white;
  border-bottom: 1px solid #eee;
  padding: 0 10px;
  box-sizing: border-box;
  display: table;
  width: 100%;
  table-layout: fixed;
}

.back-btn {
  display: table-cell;
  vertical-align: middle;
  font-size: 20px;
  font-weight: bold;
  width: 24px;
  text-align: left;
}

.title-text {
  display: table-cell;
  vertical-align: middle;
  text-align: center;
  font-size: 16px;
  font-weight: bold;
}

/* 游戏 */
.game-item {
  margin: 15px 10px;
  display: flex;
  align-items: flex-start;
}

.game-img {
  width: 80px;
  height: 70px;
  object-fit: cover;
  border-radius: 5px;
  margin-right: 10px;
}

.game-info {
  flex: 1;
}

.game-name {
  font-size: 16px;
  font-weight: bold;
  margin-top: 5px;
}

.price-box {
  margin-top: 8px;
  background-color: #f5f5f5;
  display: inline-block;
  padding: 2px 6px;
  border-radius: 3px;
  font-size: 14px;
}

.coin-icon {
  width: 16px;
  height: 16px;
  vertical-align: middle;
  margin-right: 4px;
}


/* 没有之口前的价钱 */
.old-price {
  text-decoration: line-through;
  color: #999;
  margin-left: 4px;
}
</style>