<template>
  <div class="xz_selecter">
    <div class="xz_domain">
      <div class="xz_search_input">
        <input type="text" placeholder="搜索">
      </div>
      <div class="xz_tabs">
        <ul ref="region">
          <li :class="c_region==0?'xz_choose':''">省/直辖市</li>
          <li :class="c_region==1?'xz_choose':''">城市</li>
          <li :class="c_region==2?'xz_choose':''">区/县</li>
        </ul>
      </div>
      <div class="xz_total_domain">
        <!--省选择-->
        <!-- <div class="xz_province">
          <ul>
            <li v-for="(p,index) in domain" :key="p.province.code" @click="chooseProvince(index,p)" ref="province_btn"
              :class="index == 0?'choosedProvince':''">
              {{p.province.name}}
            </li>
          </ul>
        </div> -->
        <div class="xz_region" v-show="c_region==0" ref="province_choose">
          <span class="single_city" v-for="(p,index) in domain" :key="p.province.code">
            <i @click="chooseProvince(index,p)" ref="province_btn">{{p.province.name}}</i>
          </span>
        </div>
        <!--市-->
        <div class="xz_region" v-show="c_region==1" ref="city_choose">
          <span class="single_city" v-for="city in citys" :key="city.code">
            <i>{{city.name}}</i>
            <!--多选框-->
            <!-- <span class="xz_checkbox" v-if="multipleCity">
              <input type="checkbox">
              <span class="xz_custom_checkbox"></span>
            </span> -->
          </span>
        </div>
        <!--区-->
        <div class="xz_region" v-show="c_region==2">
          <span class="single_city" v-for="area in areaList" :key="area.code">
            <i>{{area.name}}</i>
          </span>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
  import Data from './data'; //处理的数据
  export default {
    props: {
      hotCity: {
        type: Array,
        default: function () {
          return new Array()
        }
      },
      multipleCity: {
        type: Boolean,
        default: false
      }
    },
    data() {
      return {
        province: {}, //省
        city: [], //市
        area: [], //区
        domain: [
        //   {
        //   area: [],
        //   city: [],
        //   province: {
        //     name: '热门城市',
        //     code: '000000'
        //   }
        // }
        ], //整理出来的分类集合 (按省分)
        //测试的热门城市 为了更好地选择和显示统一，还是得传code 数组
        testHot: ['130300', '220300', '441400', '460100', '511900', '530800', '610100'],
        //选择的省的城市
        citys: [],
        //选择的区
        areaList:[{code:"1111",name:"hello"}],
        //选择的板块
        c_region:0
      }
    },
    created() {
      this.province = Data.province;
      this.city = Data.city;
      this.area = Data.area;
      this.count = 0;
      //热门城市 (需要监听，获取到的数据可能会延迟)
      //根据传入的热门code来筛选出热门城市
      // if (this.testHot.length > 0) {
      //   let citys = [];
      //   for (let i = 0; i < this.testHot.length; i++) {
      //     for (let ii = 0; ii < this.city.length; ii++) {
      //       if (this.testHot[i] == this.city[ii].code) {
      //         citys.push(this.city[ii]);
      //       }
      //     }
      //   }
      //   this.domain[0].city = this.citys = citys;
      // }

      for (let i = 0; i < this.province.length; i++) {
        //循环 省
        let range = {
          province: this.province[i]
        };
        //获取省前两位
        let provinceCode = this.province[i].code.substr(0, 2);
        let city = []; //筛选出的市集合
        let area = []; //筛选出的区域集合
        //获取省下的市
        for (let ii = 0; ii < this.city.length; ii++) {
          if (this.city[ii].code.substr(0, 2) == provinceCode) {
            //市的头两位跟省如果一样，那么则是在省的下面
            city.push(this.city[ii]);
          }
        }
        //如果没有拿到市，像北京省编号110000，下面没有市直接是区的数据
        //则自定义添加与省同名的市，code 后三位为100
        if (city.length == 0) {
          let code = this.province[i].code;
          range.city = [{
            name: this.province[i].name,
            code: this.province[i].code.replace(/.{3}$/, '100'), //转换最后三位
          }]
        } else {
          range.city = city;
        }
        //获取省下的区
        //规则，头两位与省头两位相同，且最后两位不为00的
        for (let a = 0; a < this.area.length; a++) {
          if (this.area[a].code.substr(0, 2) == provinceCode && (this.area[a].code.substr(this.area[a].code.length - 2,
              2) != '00')) {
            area.push(this.area[a]);
          }
        }
        range.area = area;
        this.count += area.length;
        this.domain.push(range);
      }
    },
    mounted(){
      const tabs = this.$refs.region.getElementsByTagName("li");
      const p_choose = this.$refs.province_choose.getElementsByTagName("i");
      const _this = this;
      //切换tab
      for(let i = 0; i < tabs.length; i++){
        tabs[i].addEventListener('click', function(){
          //切换
          _this.c_region = i;
        }.bind(tabs[i]),false);
      }
      //选择省份
    },
    methods: {
      //选择省
      chooseProvince(index, choice) {
        //选择项 不是已选择项， 选择项加class, 其他项有class的去掉class
        if (this.$refs.province_btn[index].classList.value.indexOf('choosedProvince') == -1) {
          //移除class
          this.removeClass(this.$refs.province_btn, 'choosedProvince');
          //选择项添加class
          this.$refs.province_btn[index].classList.add('choosedProvince');
          //province
          //拿出点击的省中的城市
          for (let i = 0; i < this.domain.length; i++) {
            if (this.domain[i].province.code == choice.province.code) {
              this.citys = this.domain[i].city;
              break;
            }
          }
          //////
          //选区切换
          this.c_region = 1;
        }
      },
      //选择市
      chooseCity(index, choice){

      },
      //移除class
      removeClass(lists, classname) {
        for(let i = 0; i < lists.length; i++){//其他项去除class
          if (lists[i].classList.value.indexOf(classname) != -1) lists[i].classList.remove(classname);
        }
      }
    }
  }
</script>
<style scoped>
  .xz_tabs {
    width: 100%;
    padding-left: 15px;
    box-sizing: border-box;
    background: #f5f5f5;
    border-bottom: 1px solid #e6e7e7;
  }

  .xz_tabs ul {
    overflow: hidden;
    padding-left: 0;
    margin: 0;
    position: relative;
    top:1px;
  }

  .xz_tabs ul li {
    float: left;
    padding: 8px 20px;
    list-style: none;
    font-size: 14px;
    line-height: 25px;
    color: #424a5e;
    cursor: pointer;
    border:1px solid transparent;
  }

  .xz_province {
    background: #f9fafb;
    height: 100%;
    overflow-y: scroll;
    width: 155px;
    box-sizing: border-box;
  }

  .xz_province ul {
    width: 100%;
    margin: 0;
    padding: 0;
  }
  .xz_choose{
    background:#fff !important;
    border-top-color:#e6e7e7 !important;
    border-left-color:#e6e7e7 !important;
    border-right-color:#e6e7e7 !important;
  }
  .xz_province li {
    width: 100%;
    box-sizing: border-box;
    list-style: none;
    cursor: pointer;
    text-align: left;
    font-size: 14px;
    line-height: 25px;
    padding: 10px 0 10px 30px;
    color: #424a5e;
    position: relative;
  }

  .xz_province .choosedProvince::after {
    content: '';
    display: block;
    position: absolute;
    left: 0;
    top: 0;
    height: 100%;
    width: 5px;
    background: #5f4b8b;
  }

  .choosedProvince {
    background: #5f4b8b;
    color:#fff;
  }

  .xz_province li:hover {
    background: #fff;
    color: #5f4b8b;
  }

  .xz_selecter {
    position: fixed;
    z-index: 9999;
    top: 0;
    right: 0;
    left: 0;
    bottom: 0;
    background: rgba(37, 40, 48, 0.7);
    display: flex;
    align-items: center;
  }

  .xz_domain {
    width: 850px;
    background: #f5f5f5;
    margin: 0 auto;
    height: 450px;
    padding: 20px 0 0 0;

  }

  .xz_search_input {
    padding: 0 20px;
    margin-bottom: 20px;
  }

  .xz_search_input input {
    width: 180px;
    font-size: 13px;
    border-radius: 30px;
    height: 30px;
    border: 1px solid #424a5e;
    outline: none;
    box-sizing: border-box;
    padding: 0 15px;
    color: #424a5e;
  }

  .xz_total_domain {
    width: 100%;
    height: 400px;
    display: flex;
  }

  .xz_region {
    width: 100%;
    height: 100%;
    overflow-y: scroll;
    padding: 0 35px;
    box-sizing: border-box;
    background: #ffffff;
  }

  .xz_region .single_city {
    display: flex;
    float: left;
    align-items: center;
    width: 25%;
    cursor: pointer;
    font-size: 14px;
    color: #424a5e;
    line-height: 25px;
    padding: 5px 0;
  }
  .xz_region .single_city i{
    font-style: normal;
    padding:3px 8px;
  }
  .xz_region .single_city:hover {
    color: #5f4b8b;
  }

  .xz_checkbox {
    position: relative;
    width: 15px;
    height: 15px;
    display: inline-block;
    margin-left: 5px;
  }

  .xz_custom_checkbox {
    position: absolute;
    width: 100%;
    height: 100%;
    border-radius: 2px;
    background: #fff;
    border: 1px solid #ddd;
    box-sizing: border-box;
  }

  .xz_checkbox input[type="checkbox"] {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: block;
    margin: 0;
    opacity: 0;
    z-index: 2;
    cursor: pointer;
  }

  .xz_checkbox input[type="checkbox"]:checked+.xz_custom_checkbox::after {
    content: '';
    position: absolute;
    left: 0;
    top: 0;

  }
</style>