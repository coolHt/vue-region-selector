<template>
  <h1>hello </h1>
</template>
<script>
import Data from './data'; //处理的数据
export default {
  data(){
    return {
      province:{},//省
      city:[],//市
      area:[],//区
      acount:[]
    }
  },
  created(){
    this.province = Data.province;
    this.city = Data.city;
    this.area = Data.area;

    for(let i = 0; i < this.province.length; i++){
      let province = {
        province: this.province[i]
      };
      let provinceCode = this.province[i].code.substr(0,2); //获取头两位
      let city = [];
      for(let ii = 0; ii < this.city.length; ii++){ //获取省下的市
        if(this.city[ii].code.substr(0,2) == provinceCode){

          city.push(this.city[ii]);
        }
      }
      if(city.length == 0){
        let code = this.province[i].code;
        province.city = [{
          name:this.province[i].name,
          code: this.province[i].code.replace(/.{3}$/,'100'), //转换最后三位
        }]
      }else{
        province.city = city;
      }
      this.acount.push(province);
    }
    console.log(this.acount);
    
  }
}
</script>
