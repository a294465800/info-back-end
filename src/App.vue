<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'app'
}
</script>

<style>
* {
  box-sizing: border-box;
}
html, body {
  height: 100%;
}
body {
  margin: 0;
  overflow: hidden;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  height: 100%;
}

.success {
  color: #67c23a;
}

.warning {
  color: #eb9e05;
}

.info {
  color: #20a0ff;
}

.normal {
  color: #666;
}
</style>
