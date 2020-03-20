<template>
  <div id="app">
    <div class="container">
      <router-view/>
    </div>
  </div>
</template>

<style lang="scss">
#app {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  width: 100vw;
  height: 100vh;
  overflow: hidden;

  .container {
    width: 100vw;
    height: 100vh;
    overflow: hidden;
  }
}



</style>
