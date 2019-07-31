<template>
  <div class="xz_selecter"></div>
</template>
<script>
import Data from './data'; //处理的数据
export default {
  data(){
    return {
      province:{},//省
      city:[],//市
      area:[],//区
      domain:[],//整理出来的分类集合 (按省分)
    }
  },
  created(){
    this.province = Data.province;
    this.city = Data.city;
    this.area = Data.area;
    this.count = 0;

    for(let i = 0; i < this.province.length; i++){
      //循环 省
      let range = {
        province: this.province[i]
      };
      //获取省前两位
      let provinceCode = this.province[i].code.substr(0,2); 
      let city = []; //筛选出的市集合
      let area = []; //筛选出的区域集合
      //获取省下的市
      for(let ii = 0; ii < this.city.length; ii++){ 
        if(this.city[ii].code.substr(0,2) == provinceCode){ 
          //市的头两位跟省如果一样，那么则是在省的下面
          city.push(this.city[ii]);
        }
      }
      //如果没有拿到市，像北京省编号110000，下面没有市直接是区的数据
      //则自定义添加与省同名的市，code 后三位为100
      if(city.length == 0){
        let code = this.province[i].code;
        range.city = [{
          name:this.province[i].name,
          code: this.province[i].code.replace(/.{3}$/,'100'), //转换最后三位
        }]
      }else{
        range.city = city;
      }
      //获取省下的区
      //规则，头两位与省头两位相同，且最后两位不为00的
      for(let a = 0; a < this.area.length; a++){
        if(this.area[a].code.substr(0,2) == provinceCode && (this.area[a].code.substr(this.area[a].code.length-2,2) != '00')){
          area.push(this.area[a]);
        }
      }
      range.area = area;
      this.count += area.length;
      this.domain.push(range);
    }

    
    
  }
}
</script>
