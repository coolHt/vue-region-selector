<template>
  <div>
    <input type="text" readonly class="trigger_select" @click="region_select = !region_select" v-model="c_value"
      placeholder="请选择区域">
    <div class="xz_selector" v-show="region_select">
      <div class="xz_domain">
        <i class="xz_close" @click="region_select = false"></i>
        <!-- <div class="xz_search_input">
          <input type="text" placeholder="搜索">
        </div> -->
        <div class="xz_tabs">
          <ul ref="region">
            <li :class="c_region==0?'xz_choose':''">省/直辖市</li>
            <li :class="c_region==1?'xz_choose':''">城市</li>
            <li :class="c_region==2?'xz_choose':''">区/县</li>
          </ul>
        </div>
        <div class="xz_total_domain">
          <!--省选择-->
          <div class="xz_region" v-show="c_region==0" ref="province_choose">
            <span class="single_city" v-for="(p,index) in domain" :key="p.province.code">
              <i @click="chooseProvince(index,p)" :data-code="p.province.code"
                ref="province_btn">{{p.province.name}}</i>
            </span>
          </div>
          <!--市-->
          <div class="xz_region" v-show="c_region==1" ref="city_choose">
            <span class="single_city" v-for="(city,index) in citys" :key="city.code">
              <i @click="chooseCity(index,city)" :data-code="city.code" ref="city_btn">{{city.name}}</i>
              <!--多选框-->
              <!-- <span class="xz_checkbox" v-if="multipleCity">
              <input type="checkbox">
              <span class="xz_custom_checkbox"></span>
            </span> -->
            </span>
          </div>
          <!--区-->
          <div class="xz_region" v-show="c_region==2">
            <span class="single_city" v-for="(area,index) in areaList" :key="area.code">
              <i @click="chooseArea(index,area)" :data-code="area.code" ref="area_btn">{{area.name}}</i>
            </span>
          </div>
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
      },
      //设置的初始化地址
      selected: {
        type: Object,
        default: function () {
          return {}
        }
      }
    },
    data() {
      return {
        //控制选择器出现 
        region_select: false,
        //输入框中的值
        c_value: '',
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
        areaList: [],
        //选择的板块
        c_region: 0,
        //选择的省所在的index, 
        provinceIndex: 0,
        //可能返回的对象
        provinceChoose: {}, //已选择的省
        cityChoose: {}, //已选择的城市
        areaChoose: {}, //已选择的城市
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
        const range = {
          province: this.province[i]
        };
        //获取省前两位
        const provinceCode = this.province[i].code.substr(0, 2);
        const city = []; //筛选出的市集合
        const area = []; //筛选出的区域集合
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
          const code = this.province[i].code;
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
    mounted() {
      const tabs = this.$refs.region.getElementsByTagName("li");
      const p_choose = this.$refs.province_choose.getElementsByTagName("i");
      const _this = this;
      //切换tab
      for (let i = 0; i < tabs.length; i++) {
        tabs[i].addEventListener('click', function () {
          //切换
          _this.c_region = i;
        }.bind(tabs[i]), false);
      }
      //如果一开始就有初始化的对象
      this.initRegion(this.selected);

    },
    watch: {
      'selected': {
        handler: function () {
          this.initRegion(this.selected);
        }
      }
    },
    methods: {
      //选择省
      chooseProvince(index, choice) {
        //给已选择的省份赋值
        this.provinceChoose = choice.province;
        const _this = this;
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
              //记录省所在的位置
              this.provinceIndex = i;
              //获取该位置下的所有城市
              this.citys = this.domain[i].city;
              break;
            }
          }
          //////
          //选区切换
          setTimeout(() => {
            _this.c_region = 1;
          }, 150)
        }
      },
      //选择市
      chooseCity(index, choice) {
        //给已选择的市赋值
        this.cityChoose = choice;
        const _this = this;
        //选择项 不是已选择项， 选择项加class, 其他项有class的去掉class
        if (this.$refs.city_btn[index].classList.value.indexOf('choosedProvince') == -1) {
          //移除class
          this.removeClass(this.$refs.city_btn, 'choosedProvince');
          //选择项添加class
          this.$refs.city_btn[index].classList.add('choosedProvince');
          //city
          //拿出点击的市中的区/县
          //已经知道是哪个省，直接去那个省所在的索引，然后再找县，前4位一样即可
          const area = this.domain[this.provinceIndex].area; //所在省的所有的area
          const consist = choice.code.substr(0, 4);
          //先清空一次
          this.areaList = [];
          area.forEach((item) => {
            if (item.code.substr(0, 4) == consist) this.areaList.push(item);
          })
          //////
          //选区切换
          setTimeout(() => {
            _this.c_region = 2;
          }, 150)
        }
      },
      //选择区
      chooseArea(index, choice) {
        //给已选择的区赋值
        this.areaChoose = choice;
        if (this.$refs.area_btn[index].classList.value.indexOf('choosedProvince') == -1) {
          //移除class
          this.removeClass(this.$refs.area_btn, 'choosedProvince');
          //选择项添加class
          this.$refs.area_btn[index].classList.add('choosedProvince');
        }
        //关闭选择器
        this.region_select = false;
        //返回所选的省/市/区
        let region = {
          province: this.provinceChoose,
          city: this.cityChoose,
          area: this.areaChoose
        }
        //给输入框赋值
        this.c_value = this.provinceChoose.name + '-' + this.cityChoose.name + '-' + this.areaChoose.name;
        //将数据返回出去
        this.$emit("cRegion", region);
      },

      //移除class
      removeClass(lists, classname) {
        for (let i = 0; i < lists.length; i++) { //其他项去除class
          if (lists[i].classList.value.indexOf(classname) != -1) lists[i].classList.remove(classname);
        }
      },
      //有初始化值进行初始化
      initRegion(region) {
        //首先拿province,拿不到数据的话就说明有误，不需要再找市和区
        if (JSON.stringify(region) != "{}") { //判断是否为空
          let hasProvince = false; //记录是否有匹配的结果
          let hasCity = false;
          let hasArea = false;
          const provincecode = region.province; //省code
          const citycode = region.city; //市code
          const areacode = region.area; //区code
          let provincename;
          let cityname;
          let areaname;

          if (provincecode) {
            for (let i = 0; i < this.domain.length; i++) {
              if (this.domain[i].province.code == provincecode) {
                hasProvince = true;
                //省名
                provincename = this.domain[i].province;
                //记录省所在的位置
                this.provinceIndex = i;
                //获取该位置下的所有城市
                this.citys = this.domain[i].city;
                break;
              }
            }
          }

          //市
          if (hasProvince) {
            if (citycode) {
              const city = this.domain[this.provinceIndex].city;
              for (let i = 0; i < city.length; i++) {
                if (city[i].code == citycode) {
                  hasCity = true;
                  //市名
                  cityname = city[i];
                  //保存所在的所有citys
                  this.citys = city;
                }
              }
            }
          }
          //区
          if (hasCity) {
            if (areacode) {
              const _this = this;
              const consist = citycode.substr(0, 4);
              let getArea = [];
              //筛选出所在的区
              this.domain[this.provinceIndex].area.forEach((item) => {
                if (item.code.substr(0, 4) == consist) getArea.push(item);
              })
              //匹配区的code
              getArea.forEach((area) => {
                if (area.code == areacode) {
                  hasArea = true;
                  //区名
                  areaname = area;
                  //设置区列表
                  _this.areaList = getArea;
                  //面板直接显示区
                  _this.c_region = 2;
                }
              })
            }
          }
          //数据都匹配到之后开始设置
          if (hasArea) {
            //首先设置输入框value
            this.c_value = provincename.name + '-' + cityname.name + '-' + areaname.name;
            //设置已选择的省市区
            this.provinceChoose = provincename;
            this.cityChoose = cityname;
            this.areaChoose = areaname;
            //设置样式
            this.$nextTick(() => {
              for (let i = 0; i < this.$refs.province_btn.length; i++) {
                if (this.$refs.province_btn[i].dataset.code == provincecode) {
                  //先删一遍class
                  this.removeClass(this.$refs.province_btn, "choosedProvince");
                  this.$refs.province_btn[i].classList.add("choosedProvince");
                }
              }
              for (let i = 0; i < this.$refs.city_btn.length; i++) {
                if (this.$refs.city_btn[i].dataset.code == citycode) {
                  //先删一遍class
                  this.removeClass(this.$refs.city_btn, "choosedProvince");
                  this.$refs.city_btn[i].classList.add("choosedProvince");
                }
              }
              for (let i = 0; i < this.$refs.area_btn.length; i++) {
                if (this.$refs.area_btn[i].dataset.code == areacode) {
                  //先删一遍class
                  this.removeClass(this.$refs.area_btn, "choosedProvince");
                  this.$refs.area_btn[i].classList.add("choosedProvince");
                }
              }
            })
          }
        }
      }
    }
  }
</script>
<style scoped>
  .trigger_select {
    width: 250px;
    height: 38px;
    box-sizing: border-box;
    padding: 0 15px;
    border-radius: 5px;
    border: 1px solid #ddd;
    outline: none;
    font-size: 14px;
    background: #fff;
    cursor: pointer;
  }

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
    top: 1px;
  }

  .xz_tabs ul li {
    float: left;
    padding: 8px 20px;
    list-style: none;
    font-size: 14px;
    line-height: 25px;
    color: #424a5e;
    cursor: pointer;
    border: 1px solid transparent;
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

  .xz_choose {
    background: #fff !important;
    border-top-color: #e6e7e7 !important;
    border-left-color: #e6e7e7 !important;
    border-right-color: #e6e7e7 !important;
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
    color: #fff;
  }

  .xz_province li:hover {
    background: #fff;
    color: #5f4b8b;
  }

  .xz_selector {
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
    position: relative;
  }

  .xz_close {
    position: absolute;
    top: 15px;
    right: 15px;
    display: block;
    width: 20px;
    height: 20px;
    background: url('../assets/close.png') no-repeat;
    background-size: 100% auto;
    cursor: pointer;
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

  .xz_region .single_city i {
    font-style: normal;
    padding: 3px 8px;
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