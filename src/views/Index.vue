<script setup>
import Footer from '/src/components/Footer.vue'
</script>

<template>
  <div>
    <router-view/>
    <div class="bottom">
      <Footer></Footer>
    </div>
  </div>
</template>



<style scoped>
.bottom{
  height: 65px;
}
</style>