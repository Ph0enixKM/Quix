<template>
  <div id="app" :class="{night}">
    <Background/>
    <img alt="Quix logo" src="./assets/logo.png">
    <Katex @night="nightMode"/>
  </div>
</template>

<script>
import Background from './components/Background.vue'
import Katex from './components/Katex.vue'

export default {
  name: 'App',
  components: {
    Katex,
    Background
  },
  data() {
    return {
      night: false
    }
  },
  methods: {
    nightMode(val) {
      console.log(val)
      this.night = val
    }
  }
}
</script>

<style>
body {
  margin: 0;
  width: 100vw;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  position: absolute;
  z-index: -10;
  width: 100vw;
  min-height: 100vh;
  background: #fff;
}
#app.night {
  background: #222;
}
</style>
