<template>
  <div id="app">
    <div id="nav">
    </div>
    <router-view/>
  </div>
  
</template>

<script>
export default {
  components: {

  },

  data: () => {
    return {
      
    }
  }
  
}
</script>

<style lang="scss">

@import url('https://fonts.googleapis.com/css2?family=Work+Sans:wght@100&display=swap');

#app {
  font-family: 'Work Sans', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
