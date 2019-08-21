<template>
  <div id="app">
    <Dispose @cRegion="getValue"></Dispose>
  </div>
</template>

<script>
import Dispose from '../src/lib/dispose';
export default {
  name: 'app',
  components:{
    Dispose
  },
  methods:{
    getValue(region){
      console.log(region);
    }
  }
}
</script>

