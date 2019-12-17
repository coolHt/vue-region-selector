<template>
  <div id="app">
    <Region :search="search" :multipleCity="true" :cityGet="true" @cRegion="hellos" :selected="selected"></Region>
  </div>
</template>
<script>
import Region from './lib/dispose';
export default {
  data(){
    return {
      search: true,
      selected: "000000"
      // selected: ["",""],
    }
  },
  
  methods:{
    hellos(data){
      console.log(data);
    }
  },
  
  components:{
    Region
  }
}
</script>

