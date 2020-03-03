<template>
  <div id="app">
    <!-- <videoDemo /> -->
    <konvaDemo />
  </div>
</template>

<script>
// import videoDemo from './components/video-demo.vue'
import konvaDemo from './components/konva-demo.vue'
// import konvaDemo from './components/demo2.vue'
export default {
  name: 'App',
  components: {
    konvaDemo
  }
};
</script>

<style>
*{
  padding: 0;
  margin: 0;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
