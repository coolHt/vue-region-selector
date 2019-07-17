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
      if(region.code.slice(-4) == '0000'){
        this.province.push(region);
      }else if(region.code.slice(-4) != '0000' && region.code.slice(-2) == '00'){ //获取市的数据
        this.city.push(region);
      }else{ //获取县的数据
         this.area.push(region);
      }
    })
    
  }
}
</script>
