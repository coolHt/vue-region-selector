<template>
  <div id="app">
    <Region :search="search"></Region>
  </div>
</template>
<script>
import Region from './lib/dispose';
export default {
  data(){
    return {
      search: true
    }
  },
  
  components:{
    Region
  }
}
</script>

