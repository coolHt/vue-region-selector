<template>
  <div id="app">
    <Region :search="search" :cityGet="true" @cRegion="hellos" :multipleCity="true" :selected="selected"></Region>
  </div>
</template>
<script>
import Region from './lib/dispose';
export default {
  data(){
    return {
      search: true,
      selected: ["330100","330200"],
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

