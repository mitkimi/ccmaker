<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
  export default {
    name: 'CCSubtitleMaker'
  }
</script>

<style>
  /* CSS */
  * {
    border: 0;
    padding: 0;
    margin: 0;
  }
  .el-textarea__inner {
    height: 100%;
  }
</style>
