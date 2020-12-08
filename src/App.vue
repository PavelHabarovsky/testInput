<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <InputUserPhone @input="returnPhone" :is-not-correct="true" :other-style-with-text="styleWithTextResidentional" :other-style-without-text="styleWithoutTextResidentional"/>
  </div>
</template>

<script>
import InputUserPhone from './components/InputUserPhone.vue'

export default {
  name: 'App',
  data(){
    return {
      phone: "",
      styleWithTextResidentional: "styleWithTextDeveloper",
      styleWithoutTextResidentional: "styleWithoutTextDeveloper"
    }
  },
  methods:{
    returnPhone(phone){
      this.phone = phone
    }
  },
  computed:{
    
  },
  components: {
    InputUserPhone
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
