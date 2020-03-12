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
@import "../src/assets/font/font.css";

* {
  margin: 0;
  padding: 0;
}

.el-autocomplete-suggestion__wrap {
  max-height: 3.75rem!important;
}

body,
html,
#app {
  user-select: none;
  width: 100%;
  height: 100%;
  font-family: BOLD;
}

.v-enter,
.v-leave-to {
  transform: translateX(-100%)
}

.v-enter-active,
.v-leave-active {
  transition: all .3s ease
}
</style>
