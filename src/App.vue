<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  background-color: #fff;
  box-sizing: border-box;
}

* {
  padding: 0;
  margin: 0;
}

#app,
html,
body {
  width: 100%;
  height: 100%;
}
</style>
