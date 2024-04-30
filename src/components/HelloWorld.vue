<script setup lang="ts">
defineProps<{
  msg: string
}>()
</script>

<template>
  <div class="greetings">
    <h1 class="green">{{ msg }}</h1>
    
    <h3>
    By <a href="https://sccube.link/" target="_blank" rel="noopener">ScaredCube</a>
    </h3>
    <!--<br>-->
    <h3 class="text-slate-300">
      *该服务目前暂时无法使用！详情请<a href="https://sccube.link/biliApi_securityControl" target="_blank">见此</a>
    </h3>
    
  </div>
</template>

<style scoped>
h1 {
  font-weight: 500;
  font-size: 2.6rem;
  position: relative;
  top: -10px;
}

h3 {
  font-size: 1.2rem;
}

.greetings h1,
.greetings h3 {
  text-align: center;
}

@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
