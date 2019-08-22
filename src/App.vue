<template>
  <div id="app">
    <Dispose @cRegion="getValue" :selected="region"></Dispose>
  </div>
</template>

<script>
import Dispose from '../src/lib/dispose';
export default {
  name: 'app',
  components:{
    Dispose
  },
  data(){
    return {
      region: {
        province: "330000",
        city: "330200",
        area: "330282",
      }
    }
  },
  mounted(){
    const _this = this;
    setTimeout( ()=> {
      _this.region = {
        province: "420000",
        city: "421300",
        area: "421381"
      }
    },5000)
  },
  methods:{
    getValue(region){
      console.log(region);
    }
  }
}
</script>

