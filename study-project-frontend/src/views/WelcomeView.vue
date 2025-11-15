<script setup>
</script>

<template>
  <div style="width: 100vw;height: 100vh;overflow: hidden;display: flex">
    <div style="flex: 1;background-color: black">
      <el-image style="width: 100%;height: 100%" fit="cover"
                src="https://images.pexels.com/photos/2387873/pexels-photo-2387873.jpeg"/>
    </div>
    <div class="welcome-title">
      <div style="font-size: 30px;font-weight: bold">欢迎来到我们的学习平台</div>
      <div style="margin-top: 10px">在这里你可以学习Java</div>
      <div style="margin-top: 5px">和同性交友</div>
    </div>
    <div class="right-card">
      <router-view v-slot="{ Component }">
        <transition name="el-fade-in-linear" mode="out-in">
          <component :is="Component"/>
        </transition>
      </router-view>
    </div>
  </div>
</template>

<style scoped>
.welcome-title {
  position: absolute;
  bottom: 30px;
  left: 30px;
  color: white;
  text-shadow: 0 0 10px black;
}

.right-card {
  width: 400px;
  z-index: 1;
  background-color: var(--el-bg-color);
}
</style>