<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <p>
      For a guide and recipes on how to configure / customize this project,<br>
      check out the
      <a href="https://cli.vuejs.org" target="_blank" rel="noopener">vue-cli documentation</a>.
    </p>

    <h3>Ecosystem</h3>
    <ul>
      <li><a href="https://router.vuejs.org" target="_blank" rel="noopener">vue-router</a></li>
      <li><a href="https://vuex.vuejs.org" target="_blank" rel="noopener">vuex</a></li>
      <li><a href="https://github.com/vuejs/vue-devtools#vue-devtools" target="_blank" rel="noopener">vue-devtools</a></li>
      <li><a href="https://vue-loader.vuejs.org" target="_blank" rel="noopener">vue-loader</a></li>
      <li><a href="https://github.com/vuejs/awesome-vue" target="_blank" rel="noopener">awesome-vue</a></li>
    </ul>
  </div>


  <div>
    <input type="range" min="0" max="100" v-model="nivel">
  </div>
  <div>
    <img class = "bateria" alt= "vue logo" :src="imagenActual">
  </div>

</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data(){
    return {
      nivel : 50
    }
  },
  computed:{
    imagenActual(){
      if (this.nivel < 20)return require('../assets/Iconsmind-Outline-Battery-0.512.png')
      if (this.nivel < 40)return require('../assets/Iconsmind-Outline-Battery-25.512.png')
      if (this.nivel < 60)return require('../assets/Iconsmind-Outline-Battery-50.512.png')
      if (this.nivel < 80)return require('../assets/Iconsmind-Outline-Battery-75.512.png')
      if (this.nivel <= 100)return require('../assets/Iconsmind-Outline-Battery-100.512.png')
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
