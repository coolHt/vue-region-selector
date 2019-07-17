<template>
  <h1>hello </h1>
</template>
<script>
import Data from './data'; //处理的数据
export default {
  data(){
    return {
      province:[],//省
      city:[],//市
      area:[],//区
    }
  },
  created(){
    //过滤省市区的数据
    Data.forEach((region, index) => { //获取省的数据
      if(region.data.slice(-4) == '0000'){
        this.province.push(region);
      }
    })
    console.log(this.province);
  }
}
</script>
