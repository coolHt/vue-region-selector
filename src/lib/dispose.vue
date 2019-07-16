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
  }
}
</script>
