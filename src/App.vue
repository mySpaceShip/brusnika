<template>
  <div id="app">
    <search-users />
  </div>
</template>

<script>
import SearchUsers from './components/search-users'

export default {
  name: 'App',
  components: {
    SearchUsers
  },

  created() {
  },
  methods: {
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
