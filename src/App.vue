<template>
  <div id="app">
    <Region :search="search" :provinceGet="true" @cRegion="hellos" :selected="data"></Region>
  </div>
</template>
<script>
import Region from './lib/dispose';
export default {
  data(){
    return {
      search: true,
      data: {
        province:"420000",
        city: "420700",
        area: "420703",
      }
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

