<template>
  <div id="app">
    <!-- <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link>
    </div>-->
     <router-view/> 
    <div class="text">ABCD</div>
  </div>
</template>

<script>
  // import {mapGetters} from 'vuex'
  export default {
    computed: {
      // ...mapGetters(['test'])
    },
    mounted () {
      // this.$store.dispatch('setTest',100).then(()=>{
      //   // console.log(this.$store.state.book.test)
      //   console.log(this.test)
      // })
    }
  }

  document.addEventListener('DOMContentLoaded',()=>{
    const html = document.querySelector('html')
    let fontSize = window.innerWidth/10
    fontSize = fontSize>50?50:fontSize
    html.style.fontSize = fontSize + 'px'
  })


</script>


<style lang="scss" scoped>
@import "./assets/styles/global.scss";
.text {
  font-size: px2rem(20)
}

</style>
