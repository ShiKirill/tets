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

<style>
body {
  margin: 0;
  overflow: hidden;
}
#app {
  width: 100vw;
  height: 100vh;
  min-width: 50rem;
  /* min-height: 26rem; */
  display: flex;
}
.img-block {
  width: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.main-img {
  width: 87.99%;
  margin-top: -13px;
}
</style>
