<template>
  <div id="app">
    <div class="header">
      <router-link to="/">
        <img id="logo" src="../static/monster_logo.gif"/>
      </router-link>
    </div>
    <router-view/>
    <footer class="footer">
      <p>
        MONSTER made possible by grants from the National Cancer Institute and Northwestern University
      </p>
      <p>
        &copy; 2003-{{getYear()}} Northwestern University
      </p>
    </footer>
  </div>
</template>

<script>
export default {
  name: 'App',
  methods: {
    getYear: function () {
      return new Date().getFullYear()
    }
  }
}
</script>

<style>
  #logo{
    display: block;
    margin-left: auto;
    margin-right: auto;
  }
  .header{
    padding-bottom: 40px;
    padding-top: 40px;
  }
  .footer{
    position: fixed;
    left: 0;
    bottom: 0;
    width: 100%;
  }
</style>
