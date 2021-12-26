<template>
  <div id="app">
    <div class="img-block">
      <img alt="logo" class="main-img" src="./assets/books.png" />
    </div>
    <Form />
  </div>
</template>

<script>
import Form from './components/Form.vue';

export default {
  name: 'App',
  components: {
    Form,
  },
};
</script>

<style lang="scss">
@import url('./assets/fonts/Geometria/stylesheet.css');
body {
  margin: 0;
  overflow-x: hidden;
  font-family: Geometria, sans-serif;

  #app {
    width: 100vw;
    height: 100vh;
    min-width: 50rem;
    min-height: 26rem;
    display: flex;
  }

  .img-block {
    width: 50%;
    display: flex;
    justify-content: center;
    align-items: center;

    .main-img {
      width: 87.99%;
      margin-top: -13px;
    }
  }
}

@media screen and (min-width: 1800px) {
  html {
    font-size: 120%;
  }
}

@media screen and (max-width: 800px) {
  html {
    font-size: 90%;
  }

  .img-block {
    display: none !important;
  }

  #app {
    // height: auto !important;
    min-width: 0 !important;
  }
}

@media screen and (max-width: 400px) {
  html {
    font-size: 80%;
  }
}
</style>
