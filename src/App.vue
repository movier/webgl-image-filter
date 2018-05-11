<template>
  <div id="app">
    <h1>Getting Started with WebGL and Image Filter</h1>
    <HelloWebGL />
  </div>
</template>

<script>
import HelloWebGL from './components/HelloWebGL.vue'

export default {
  name: 'app',
  components: {
    HelloWebGL
  }
}
</script>

<style>
body {
  margin: 0;
}
h1 {
  margin: 1em 0;
  font-size: 28px;
}
#app {
  font-family: Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
