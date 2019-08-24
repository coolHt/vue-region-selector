<template>
  <div>
    <div class="xz_con">
      <i v-if="c_value.trim() != ''" class="delete_value" @click="clear"></i>
      <input type="text" readonly class="trigger_select" @click="region_select = !region_select" v-model="c_value"
        placeholder="请选择区域">
    </div>
    <div class="xz_selector" v-show="region_select" ref="selector" @click="showResult = false">
      <div class="xz_domain">
        <i class="xz_close" @click="xz_close"></i>
        <div v-if="search" class="xz_search_input">
          <input type="text" placeholder="搜索" v-model="searchField" @input="searchMethod" @click.stop="">
          <div class="search_result_con" v-show="showResult" @click.stop="">
            <div class="search_result">
              <div class="search_line" v-if="searchProvince.length > 0">
                <h1>省/直辖市</h1>
                <div class="search_item_con">
                  <div class="search_item" v-for="(province) in searchProvince">
                    <span ref="sProvince" @click="sProvince(province.code)">{{province.name}}</span>
                  </div>
                </div>
              </div>
              <div class="search_line" v-if="searchCity.length > 0 && showCityTab">
                <h1>城市</h1>
                <div class="search_item_con">
                  <div class="search_item" v-for="(city) in searchCity">
                    <span ref="sCity" @click="sCity(city.code)">{{city.name}}</span>
                  </div>
                </div>
              </div>
              <div class="search_line" v-if="searchArea.length > 0 && showAreaTab">
                <h1>区/县</h1>
                <div class="search_item_con">
                  <div class="search_item" v-for="(area) in searchArea">
                    <span ref="sArea" @click="sArea(area.code)">{{area.name}}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="xz_tabs">
          <ul ref="region">
            <li :class="c_region==0?'xz_choose':''" v-if="">省/直辖市</li>
            <li :class="c_region==1?'xz_choose':''" v-if="showCityTab">城市</li>
            <li :class="c_region==2?'xz_choose':''" v-if="showAreaTab">区/县</li>
          </ul>
        </div>
        <div class="xz_total_domain">
          <!--省选择-->
          <div class="xz_region" v-show="c_region==0" ref="province_choose">
            <span class="single_city" v-for="(p,index) in province" :key="p.code">
              <i @click="chooseProvince(index,p)" :data-code="p.code" ref="province_btn">{{p.name}}</i>
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
      },
      //是否有搜索框
      search: {
        type: Boolean,
        default: false
      },
      //省市区
      provinceGet: {
        type: Boolean,
        default: false
      },
      cityGet: {
        type: Boolean,
        default: false,
      },
      areaGet: {
        type: Boolean,
        default: true
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
        domain: [], //整理出来的分类集合 (按省分)
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
        //搜索的字段
        searchField: '',
        //搜索结果
        searchProvince: [],
        searchCity: [],
        searchArea: [],
        //显示搜索结果
        showResult: false
      }
    },
    created() {
      this.province = Data.province;
      this.city = Data.city;
      this.area = Data.area;
      //热门城市 (需要监听，获取到的数据可能会延迟)

      //按省整合数据
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
        this.domain.push(range);
      }
    },
    mounted() {
      const tabs = this.$refs.region.getElementsByTagName("li");
      const _this = this;
      //模式切换，如果有设置多选项，以最低级为准，比如，同时设置了市和区的选板，就以区为准 //provinceGet cityGet areaGet

      if (this.areaGet) { //选板为 区域

      } else if (this.cityGet) { //选板为 城市
        //需要改变模板吗

      } else if (this.provinceGet) { //选板为 省份
        //需要改变模板吗
      }
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
    computed: {
      showAreaTab() { //控制显示区tab
        if (this.provinceGet || this.cityGet) { //如果设置了省或市的其中一个
          return false;
        } else {
          return true
        }
      },
      showCityTab() { //控制显示城市tab
        if (this.provinceGet) { //如果设置了省
          return false;
        } else {
          return true;
        }
      }
    },
    methods: {
      //选择省
      chooseProvince(index, choice) {
        //给已选择的省份赋值
        this.provinceChoose = choice;
        const _this = this;
        //选择项 不是已选择项， 选择项加class, 其他项有class的去掉class
        if (this.$refs.province_btn[index].classList.value.indexOf('choosedProvince') == -1) {
          this.areaList = [];
          //移除class
          this.removeClass(this.$refs.province_btn, 'choosedProvince');
          //选择项添加class
          this.$refs.province_btn[index].classList.add('choosedProvince');
          //province
          //拿出点击的省中的城市
          for (let i = 0; i < this.domain.length; i++) {
            if (this.domain[i].province.code == choice.code) {
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
            //差别
            if (_this.provinceGet) { //如果设置了选省，直接弹出
              //关闭选择器
              _this.region_select = false;
              //返回所选的省/市/区
              _this.returnRegion();
            } else { //否则进入下一个
              _this.c_region = 1;
            }
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
            //差别
            if (_this.cityGet) { //已经过滤掉省，所以只需要判断有没有设置市
              //关闭选择器
              _this.region_select = false;
              //返回所选的省/市/区
              _this.returnRegion();
            } else { //如果没有设置市
              _this.c_region = 2;
            }

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
        this.returnRegion();
      },

      //有初始化值进行初始化
      initRegion(region) {
        //首先拿province,拿不到数据的话就说明有误，不需要再找市和区
        if (JSON.stringify(region) != "{}") { //判断是否为空
          //记录是否有匹配的结果
          let hasProvince = false;
          let hasCity = false;
          let hasArea = false;
          //
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
          //需要判断是否需要执行 (如果设置了city 并且 没有设置province的情况下)
          //市
          if (hasProvince && (!this.provinceGet)) {
            if (citycode) {
              const city = this.domain[this.provinceIndex].city;
              for (let i = 0; i < city.length; i++) {
                if (city[i].code == citycode) {
                  if (this.cityGet) { //如果设置了城市，则跳到城市板块
                    this.c_region = 1;
                  }
                  hasCity = true;
                  //市名
                  cityname = city[i];
                  //保存所在的所有citys
                  this.citys = city;
                  break;
                }
              }
            }
          }
          //需要判断是否需要执行 (如果city 和 province 都没有设置)
          //区
          if (hasCity && (!this.provinceGet) && (!this.cityGet)) {
            if (areacode) {
              const _this = this;
              const consist = citycode.substr(0, 4);
              let getArea = [];
              //筛选出所在的区
              this.domain[this.provinceIndex].area.forEach((item) => {
                if (item.code.substr(0, 4) == consist) getArea.push(item);
              })
              //匹配区的code
              for (let i = 0; i < getArea.length; i++) {
                if (getArea[i].code == areacode) {
                  hasArea = true;
                  //区名
                  areaname = getArea[i];
                  //设置区列表
                  _this.areaList = getArea;
                  //面板直接显示区
                  _this.c_region = 2;
                  break;
                }
              }
            }
          }
          //数据都匹配到之后开始设置
          if (hasProvince && (!hasArea) && (!hasCity)) { //设置到省
            this.c_value = provincename.name;
            this.provinceChoose = provincename;
            this.$nextTick(() => {
              for (let i = 0; i < this.$refs.province_btn.length; i++) {
                if (this.$refs.province_btn[i].dataset.code == provincecode) {
                  //先删一遍class
                  this.removeClass(this.$refs.province_btn, "choosedProvince");
                  this.$refs.province_btn[i].classList.add("choosedProvince");
                  break;
                }
              }
            })
          }
          if (hasCity && (!hasArea)) { //设置到市
            this.c_value = provincename.name + '-' + cityname.name;
            this.provinceChoose = provincename;
            this.cityChoose = cityname;
            this.$nextTick(() => {
              for (let i = 0; i < this.$refs.province_btn.length; i++) {
                if (this.$refs.province_btn[i].dataset.code == provincecode) {
                  //先删一遍class
                  this.removeClass(this.$refs.province_btn, "choosedProvince");
                  this.$refs.province_btn[i].classList.add("choosedProvince");
                  break;
                }
              }
              for (let i = 0; i < this.$refs.city_btn.length; i++) {
                if (this.$refs.city_btn[i].dataset.code == citycode) {
                  //先删一遍class
                  this.removeClass(this.$refs.city_btn, "choosedProvince");
                  this.$refs.city_btn[i].classList.add("choosedProvince");
                  break;
                }
              }
            })
          }
          if (hasArea) { //设置到区
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
                  break;
                }
              }
              for (let i = 0; i < this.$refs.city_btn.length; i++) {
                if (this.$refs.city_btn[i].dataset.code == citycode) {
                  //先删一遍class
                  this.removeClass(this.$refs.city_btn, "choosedProvince");
                  this.$refs.city_btn[i].classList.add("choosedProvince");
                  break;
                }
              }
              for (let i = 0; i < this.$refs.area_btn.length; i++) {
                if (this.$refs.area_btn[i].dataset.code == areacode) {
                  //先删一遍class
                  this.removeClass(this.$refs.area_btn, "choosedProvince");
                  this.$refs.area_btn[i].classList.add("choosedProvince");
                  break;
                }
              }
            })
          }
        }
      },
      //关闭
      xz_close() {
        this.region_select = false;
        //初始化
        //this.clear();
      },
      //初始化,清除所有已选择的数据
      clear() {
        //已选择的
        this.provinceChoose = {};
        this.cityChoose = {};
        this.areaChoose = {};
        this.c_value = ''; //清空文字
        this.c_region = 0; //面板位置恢复
        this.searchField = '';
        //搜索结果
        this.searchProvince = [];
        this.searchCity = [];
        this.searchArea = [];
        //省的数据一直都存在所以不需要清只需要清理class
        this.removeClass(this.$refs.province_btn, "choosedProvince");
        //选择的省的城市
        this.citys = [];
        //选择的区
        this.areaList = [];
      },
      //赋值选择的地区并返回出数据
      returnRegion() {
        let region;
        if ((!this.provinceGet) && (!this.cityGet)) { //如果省、市都没有设置
          region = {
            province: this.provinceChoose,
            city: this.cityChoose,
            area: this.areaChoose
          }
          //给输入框赋值
          this.c_value = this.provinceChoose.name + '-' + this.cityChoose.name + '-' + this.areaChoose.name;
        } else if ((!this.provinceGet) && this.cityGet) { //如果设置了市并且没有设置省
          region = {
            province: this.provinceChoose,
            city: this.cityChoose
          }
          //给输入框赋值
          this.c_value = this.provinceChoose.name + '-' + this.cityChoose.name;
        } else if (this.provinceGet) { //如果设置了省
          region = {
            province: this.provinceChoose
          }
          //给输入框赋值
          this.c_value = this.provinceChoose.name;
        }

        //将数据返回出去
        this.$emit("cRegion", region);
      },
      //移除class
      removeClass(lists, classname) {
        for (let i = 0; i < lists.length; i++) { //其他项去除class
          if (lists[i].classList.value.indexOf(classname) != -1) lists[i].classList.remove(classname);
        }
      },
      //搜索
      searchMethod() {
        if (this.searchField.trim() != '') {
          //初始化
          this.searchProvince = [];
          this.searchCity = [];
          this.searchArea = [];
          if ((!this.provinceGet) && (!this.cityGet)) { //没有设置市和省
            //搜索区
            for (let i = 0; i < this.area.length; i++) {
              if (this.area[i].name.indexOf(this.searchField) != -1) {
                this.searchArea.push(this.area[i]);
              }
            }
          }

          if (!this.provinceGet) { //如果没有设置省
            //搜索市
            for (let i = 0; i < this.city.length; i++) {
              if (this.city[i].name.indexOf(this.searchField) != -1) {
                this.searchCity.push(this.city[i]);
              }
            }
          }

          //搜索省
          for (let i = 0; i < this.province.length; i++) {
            if (this.province[i].name.indexOf(this.searchField) != -1) {
              this.searchProvince.push(this.province[i]);
            }
          }

          if (this.searchProvince.length > 0 || this.searchCity.length > 0 || this.searchArea.length > 0) this
            .showResult = true;

        } else {
          this.showResult = false;
          this.searchProvince = [];
          this.searchCity = [];
          this.searchArea = [];
        }
      },
      //选择搜索结果
      sProvince(code) {
        this.c_region = 0;
        //还是要还原选项
        this.areaList = [];
        this.citys = [];
        this.showResult = false;
        this.searchField = '';
        //
        const code2 = code.substr(0, 2); //通过这个找省
        for (let i = 0; i < this.province.length; i++) {
          if (this.province[i].code.substr(0, 2) == code2) {
            this.provinceChoose = this.province[i]; //设置选中的省
            this.removeClass(this.$refs.province_btn, "choosedProvince");
            this.$refs.province_btn[i].classList.add("choosedProvince");
            //知道了是哪个省之后就可以拿市了
            this.citys = this.domain[i].city;
            if (this.provinceGet) { //如果设置了省，直接返回
              //关闭选择框
              this.region_select = false;
              this.returnRegion();
            }
            break;
          }
        }
      },
      sCity(code) {
        this.c_region = 1;
        //还是要还原选项
        this.areaList = [];
        this.citys = [];
        this.showResult = false;
        this.searchField = '';
        //
        const code2 = code.substr(0, 2); //通过这个找省
        const code4 = code.substr(0, 4); //通过这个找市
        let provinceIndex;
        for (let i = 0; i < this.province.length; i++) {
          if (this.province[i].code.substr(0, 2) == code2) {
            this.provinceChoose = this.province[i]; //设置选中的省
            this.removeClass(this.$refs.province_btn, "choosedProvince");
            this.$refs.province_btn[i].classList.add("choosedProvince");
            provinceIndex = i; //记录哪个省
            //知道了是哪个省之后就可以拿市了
            this.citys = this.domain[i].city;
            this.$nextTick(() => {
              for (let i = 0; i < this.citys.length; i++) {
                if (this.citys[i].code.substr(0, 4) == code4) {
                  this.cityChoose = this.citys[i]; //设置选中的市
                  this.removeClass(this.$refs.city_btn, "choosedProvince");
                  this.$refs.city_btn[i].classList.add("choosedProvince");

                  if (this.cityGet) { //如果设置了市，直接返回
                    //关闭选择框
                    this.region_select = false;
                    this.returnRegion();
                  }

                  //筛选区/县 (如果没有设置省和城市才需要)
                  if ((!this.provinceGet) && (!this.cityGet)) {
                    this.domain[provinceIndex].area.forEach((item) => {
                      if (item.code.substr(0, 4) == code4) this.areaList.push(item);
                    })
                  }

                  break;
                }
              }
            })
            break;
          }
        }
      },
      sArea(code) { //已经是区了，所以直接关闭选择
        this.showResult = false;
        this.c_region = 2;
        this.searchField = '';
        //关闭选择框
        this.region_select = false;
        //还是要还原选项
        this.areaList = [];
        this.citys = [];

        //但是还是要找到关联的数据
        const code2 = code.substr(0, 2); //通过这个找省
        const code4 = code.substr(0, 4); //通过这个找市
        let provinceIndex;
        for (let i = 0; i < this.province.length; i++) {
          if (this.province[i].code.substr(0, 2) == code2) {
            this.provinceChoose = this.province[i]; //设置选中的省
            this.removeClass(this.$refs.province_btn, "choosedProvince");
            this.$refs.province_btn[i].classList.add("choosedProvince");
            provinceIndex = i; //记录哪个省
            //知道了是哪个省之后就可以拿市了
            this.citys = this.domain[i].city;
            this.$nextTick(() => {
              for (let i = 0; i < this.citys.length; i++) {
                if (this.citys[i].code.substr(0, 4) == code4) {
                  this.cityChoose = this.citys[i]; //设置选中的市
                  this.removeClass(this.$refs.city_btn, "choosedProvince");
                  this.$refs.city_btn[i].classList.add("choosedProvince");
                  //筛选区/县
                  this.domain[provinceIndex].area.forEach((item) => {
                    if (item.code.substr(0, 4) == code4) this.areaList.push(item);
                  })
                  for (let i = 0; i < this.areaList.length; i++) {
                    if (this.areaList[i].code == code) {
                      this.areaChoose = this.areaList[i]; //设置选中的区
                      this.$nextTick(() => {
                        this.removeClass(this.$refs.area_btn, "choosedProvince");
                        this.$refs.area_btn[i].classList.add("choosedProvince");

                      })
                      //返回数据
                      this.returnRegion();
                      break;
                    }
                  }
                  break;
                }
              }
            })
            break;
          }
        }
      }
    }
  }
</script>
<style scoped>
  .xz_con {
    width: 270px;
    position: relative;
  }

  .search_line {
    width: 100%;
  }

  .search_line h1 {
    font-size: 14px;
    padding: 5px 0 5px 10px;
    margin: 0 0 5px 0;
    width: 100%;
    box-sizing: border-box;
    border-bottom: 1px solid #ddd;
  }

  .search_item_con {
    width: 90%;
    margin: 0 auto 10px auto;
    display: flex;
    flex-wrap: wrap;
  }

  .search_item {
    width: 33%;
  }

  .search_item span {
    padding: 5px 5px;
    margin-bottom: 5px;
    font-size: 13px;
    display: inline-block;
    white-space: nowrap;
    cursor: pointer;
  }

  .search_item span:hover {
    background: #5f4b8b;
    color: #fff;
  }

  .delete_value {
    position: absolute;
    right: 15px;
    top: 15px;
    display: block;
    width: 13px;
    height: 13px;
    cursor: pointer;
  }

  .delete_value::before {
    position: absolute;
    content: '';
    display: block;
    width: 13px;
    height: 2px;
    transform-origin: 50% 50%;
    background: #424a5e;
    left: 0px;
    top: 7px;
    border-radius: 20px;
    transform: rotatez(130deg);
  }

  .delete_value::after {
    position: absolute;
    content: '';
    display: block;
    width: 13px;
    height: 2px;
    transform-origin: 50% 50%;
    background: #424a5e;
    left: 0px;
    top: 7px;
    border-radius: 20px;
    transform: rotatez(50deg);
  }

  .trigger_select {
    width: 100%;
    height: 45px;
    box-sizing: border-box;
    padding: 0 15px;
    border-radius: 5px;
    border: 1px solid #ddd;
    outline: none;
    font-size: 14px;
    background: #fff;
    cursor: pointer;
    color: #424a5e;
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
    cursor: pointer;
    z-index: 100;
  }

  .xz_close::after {
    content: '';
    width: 20px;
    height: 2px;
    display: block;
    background: #424a5e;
    position: absolute;
    left: 0;
    top: 3px;
    transform-origin: 100%;
    transform: rotatez(-45deg);
  }

  .xz_close::before {
    content: '';
    width: 20px;
    height: 2px;
    display: block;
    background: #424a5e;
    position: absolute;
    right: -6px;
    top: 3px;
    transform-origin: 0;
    transform: rotatez(45deg);
  }

  .xz_search_input {
    padding: 0 20px;
    margin-bottom: 20px;
    position: relative;
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

  .xz_search_input .search_result_con {
    position: absolute;
    left: 15px;
    top: 45px;
    z-index: 200;
  }

  .xz_search_input .search_result {
    min-width: 200px;
    max-width: 700px;
    min-height: 100px;
    max-height: 250px;
    overflow-y: auto;
    border: 1px solid #ddd;
    background: #fff;
  }

  .xz_search_input .search_result_con::before {
    content: '';
    display: block;
    width: 0;
    height: 0;
    position: absolute;
    left: 80px;
    top: -19px;
    border: 10px solid #fff;
    border-top: 10px solid transparent;
    border-left-color: transparent;
    border-right-color: transparent;
  }

  .xz_total_domain {
    width: 100%;
    height: 400px;
    display: flex;
  }

  .xz_region {
    width: 100%;
    height: 100%;
    overflow-y: auto;
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