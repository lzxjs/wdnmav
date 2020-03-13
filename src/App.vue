<template>
  <div id="app">
    <transition mode="out-in" appear>
      <router-view/>
    </transition>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
}

/* @font-face {
  font-family: "BOLD";
  src: url('https://cdn.92lzx.cn/ziti.otf');
}

body,
html,
#app {
  user-select: none;
  width: 100%;
  height: 100%;
  font-family: BOLD;
} */

.v-enter,
.v-leave-to {
  transform: translateX(-100%)
}

.v-enter-active,
.v-leave-active {
  transition: all .3s ease
}
</style>
