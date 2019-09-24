<template>
  <div>
    <div class="xz_con">
      <i v-if="c_value.trim() != ''" class="delete_value" @click="clearReturn"></i>
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
                  <div class="search_item" v-for="(province) in searchProvince" :key="province.code">
                    <span ref="sProvince" @click="sProvince(province.code)">{{province.name}}</span>
                  </div>
                </div>
              </div>
              <div class="search_line" v-if="searchCity.length > 0 && showCityTab">
                <h1>城市</h1>
                <div class="search_item_con">
                  <div class="search_item" v-for="(city) in searchCity" :key="city.code">
                    <span ref="sCity" @click="sCity(city.code)">{{city.name}}</span>
                  </div>
                </div>
              </div>
              <div class="search_line" v-if="searchArea.length > 0 && showAreaTab">
                <h1>区/县</h1>
                <div class="search_item_con">
                  <div class="search_item" v-for="(area) in searchArea" :key="area.code">
                    <span ref="sArea" @click="sArea(area.code)">{{area.name}}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="xz_tabs">
          <!--多选按钮-->
          <button class="multpleReturn" @click="multpleReturn"
            v-if="!provinceGet && cityGet && multipleCity">选择完成</button>
          <!--单选无区域差按钮-->
          <button class="multpleReturn" @click="returnSingle"
            v-if="fullrange && !multipleCity">选择完成</button> <!--multipleCity 不能为true-->
          <button class="clearChoose" @click="clear" v-if="!provinceGet && cityGet && multipleCity">清除已选择</button>
          <ul ref="region">
            <li :class="c_region==0?'xz_choose':''">省/直辖市</li>
            <li :class="c_region==1?'xz_choose':''" v-if="showCityTab">城市</li>
            <li :class="c_region==2?'xz_choose':''" v-if="showAreaTab">区/县</li>
            <li v-if="fullrange && !multipleCity">
              已选择: 
              <span v-if="isEmptyObj(provinceChoose)">{{provinceChoose.name}}</span>
              <span v-if="isEmptyObj(cityChoose)"> / {{cityChoose.name}}</span>
              <span v-if="isEmptyObj(areaChoose)"> / {{areaChoose.name}}</span>
            </li>
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
              <!-- <span class="xz_checkbox" v-if="!provinceGet && cityGet && multipleCity">
                <input type="checkbox" ref="chooseBox">
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
        // 默认false
        //如果开启了，搜索条件下选城市不直接完成，需要显示城市点击完成才行
        type: Boolean,
        default: false
      },
      //设置的初始化地址
      selected: '',
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
      },
      //设置无区域范围单选
      fullrange:{  //为true表示可无差别选择单个区域省或市或区都行
        type: Boolean,
        default: false
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
        cityChooseArr: [], //已选择的城市数组
        areaChoose: {}, //已选择的城市
        //搜索的字段
        searchField: '',
        //搜索结果
        searchProvince: [],
        searchCity: [],
        searchArea: [],
        //显示搜索结果
        showResult: false,
        //无区域限制情况下判断选到了哪一区域阶段
        rangeSet:0
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
          if(i == 3) return;
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
        if (this.provinceChoose.code == choice.code) { //如果选的是同一个，没必要进行下去了
          return;
        }

        //因为选择的不是同一个省，所以已选择的要重置
        this.cityChoose = {}; //已选择的城市
        this.cityChooseArr = []; //已选择的城市数组
        this.areaChoose = {}; //已选择的城市
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
        if (this.multipleCity && this.cityGet) { //如果多选
        } else {
          if(choice.code != this.cityChoose.code){
            this.areaChoose = {};
          }
          this.cityChoose = choice;
        }
        const _this = this;
        //选择项 不是已选择项， 选择项加class, 其他项有class的去掉class
        if (this.$refs.city_btn[index].classList.value.indexOf('choosedProvince') == -1 || this.multipleCity) {
          if (!this.cityGet || this.fullrange) { //如果不是选择城市
            //移除class
            this.removeClass(this.$refs.city_btn, 'choosedProvince');
            //选择项添加class
            this.$refs.city_btn[index].classList.add('choosedProvince');
          } else if (this.cityGet && this.multipleCity) { //可以点多个
            if (this.$refs.city_btn[index].classList.contains('choosedProvince')) {
              this.$refs.city_btn[index].classList.remove('choosedProvince');
              //去掉数组中存在的
              this.cityChooseArr.forEach((city, index) => {
                if (city.code == choice.code) {
                  this.cityChooseArr.splice(index, 1);
                }
              })
            } else {
              this.$refs.city_btn[index].classList.add('choosedProvince');
              //判断原来的数组中有没有存在
              this.cityChooseArr.push(choice);
            }
          }

          //先清空一次
          this.areaList = [];
          if (!_this.cityGet && !_this.provinceGet) { //如果要选择区的话
            //city
            //拿出点击的市中的区/县
            //已经知道是哪个省，直接去那个省所在的索引，然后再找县，前4位一样即可
            const area = this.domain[this.provinceIndex].area; //所在省的所有的area
            const consist = choice.code.substr(0, 4);
          
            area.forEach((item) => {
              if (item.code.substr(0, 4) == consist) this.areaList.push(item);
            })
            
          }
          //////
          //选区切换
          setTimeout(() => {
            //差别
            if (_this.cityGet) { //已经过滤掉省，所以只需要判断有没有设置市
              //返回所选的省/市/区
              if (!_this.multipleCity) { //如果不是多选
                if(!this.fullrange){
                  _this.region_select = false;
                  _this.returnRegion();
                  //关闭选择器
                }
              }
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
        if(!this.fullrange){  //如果fullrange为false的话，直接返回并关闭数据
           //关闭选择器
          this.region_select = false;
           //返回所选的省/市/区
          this.returnRegion();
        }
      },

      //有初始化值进行初始化
      initRegion(region) {
        //首先拿province,拿不到数据的话就说明有误，不需要再找市和区
        if (this.selected) { //判断是否为空
          //需要判断值是不是数组，并且是属于什么类型的
          let codeData = '';
          let isProvince = false;
          let isCity = false;
          let isArea = false;
          let provincename;
          let cityname;
          let citynameArr = []; //多选数组
          let areaname;
          //如果是数组
          Array.isArray(this.selected) ? codeData = this.selected[0] : codeData = this.selected;
          //需要判断code 是省的还是市的还是区的
          if (codeData.slice(-4) == '0000') { //省
            isProvince = true;
          } else if (codeData.slice(-4) != '0000' && codeData.slice(-2) == '00') { //是市
            isCity = true;
          } else { //区
            isArea = true;
          }
          //根据模式的不同来设置,如果模式与code的类型不符合就不用去匹对了
          if (isProvince && (this.provinceGet || this.fullrange)) { //如果是省类型的code 并且是省模式
            for (let i = 0; i < this.domain.length; i++) {
              if (this.domain[i].province.code == codeData) {
                //省名
                provincename = this.domain[i].province;
                this.provinceChoose = provincename;
                if(this.fullrange && !this.provinceGet){
                  //获取该位置下的所有城市
                  this.citys = this.domain[i].city;
                }
                //记录省所在的位置
                this.provinceIndex = i;
                break;
              }
            }
            //设置
            this.$nextTick(() => {
              for (let i = 0; i < this.$refs.province_btn.length; i++) {
                if (this.$refs.province_btn[i].dataset.code == codeData) {
                  //先删一遍class
                  this.removeClass(this.$refs.province_btn, "choosedProvince");
                  this.$refs.province_btn[i].classList.add("choosedProvince");
                  this.c_value = provincename.name;
                  break;
                }
              }
            })
          }
          if (isCity && ((this.cityGet && !this.provinceGet) || this.fullrange)) { //如果是市类型的code 并且是市模式 或者是跨区单选模式
            for (let i = 0; i < this.domain.length; i++) {
              let code2 = codeData.substr(0, 2); //匹配区
              if (this.domain[i].province.code.substr(0, 2) == code2) {
                //省名
                provincename = this.domain[i].province;
                //获取该位置下的所有城市
                this.citys = this.domain[i].city;
                this.provinceChoose = provincename;
                //记录省所在的位置
                this.provinceIndex = i;
                
                break;
              }
            }
            //判断是多选还是单选
            let isMultple = Array.isArray(this.selected);
            if (isMultple && this.multipleCity) { //数组并且多选
              let _this = this;
              let hasCity = false;
              let indexArr = [];
              this.selected.forEach((city) => {
                for (let c = 0; c < _this.citys.length; c++) {
                  if (city == _this.citys[c].code) {
                    if (!hasCity) hasCity = true; //
                    indexArr.push(c);
                  }
                }
              })
              if (hasCity) {
                this.c_region = 1;
                this.$nextTick(() => {
                  //先删一遍class
                  let nameList = '';
                  this.removeClass(this.$refs.province_btn, "choosedProvince");
                  this.removeClass(this.$refs.city_btn, "choosedProvince");
                  //设置省
                  this.$refs.province_btn[this.provinceIndex].classList.add("choosedProvince");
                  this.cityChooseArr = []; //先清空吧
                  indexArr.forEach((arr) => {
                    _this.$refs.city_btn[arr].classList.add("choosedProvince");
                    _this.cityChooseArr.push(_this.citys[arr]);
                    if (arr != indexArr.length - 1) {
                      nameList += `${_this.citys[arr].name}, `;
                    } else {
                      nameList += `${_this.citys[arr].name}`;
                    }

                    this.c_value = `${provincename.name} (${nameList})`;

                  })
                })
              }
            } else {
              for (let c = 0; c < this.citys.length; c++) {
                if (this.citys[c].code == codeData) {
                  this.cityChoose = this.citys[c];
                  this.$nextTick(() => {
                    //先删一遍class
                    this.removeClass(this.$refs.province_btn, "choosedProvince");
                    this.removeClass(this.$refs.city_btn, "choosedProvince");
                    //设置省
                    this.$refs.province_btn[this.provinceIndex].classList.add("choosedProvince");
                    //设置市
                    this.$refs.city_btn[c].classList.add("choosedProvince");
                    this.c_region = 1;

                    this.c_value = `${provincename.name}-${this.citys[c].name}`;
                  })
                  break;
                }
              }
            }
          }
          if (isArea && !this.cityGet && !this.provinceGet) { //如果是区类型的code,并且是区模式
            let code2 = codeData.substr(0, 2); //匹配区
            let code4 = codeData.substr(0, 4); //匹配到市
            let drawA = 0; //渲染区域的index
            let drawC = 0; //渲染市的index
            for (let i = 0; i < this.domain.length; i++) {
              if (this.domain[i].province.code.substr(0, 2) == code2) {
                //省名
                provincename = this.domain[i].province;
                this.provinceChoose = provincename;
                //记录省所在的位置
                this.provinceIndex = i;
                //获取该位置下的所有城市
                this.citys = this.domain[i].city;
                //如果当前的省下的区中有匹配的
                let areaArr = [];
                for (let a = 0; a < this.domain[i].area.length; a++) { //拿到匹配到的区域集合
                  if (this.domain[i].area[a].code.substr(0, 4) == code4) {
                    areaArr.push(this.domain[i].area[a]);
                  }
                }
                for (let c = 0; c < areaArr.length; c++) { //这个c也就是要给样式的那个index
                  if (areaArr[c].code == codeData) { //如果能匹配到同一个区域
                    //将匹配到的区域数组赋值出去
                    this.areaList = areaArr;
                    this.areaChoose = areaArr[c];
                    for (let c2 = 0; c2 < this.citys.length; c2++) {
                      if (this.citys[c2].code.substr(0, 4) == code4) {
                        this.cityChoose = this.citys[c2];
                        drawC = c2;
                        break;
                      }
                    }
                    drawA = c;
                    this.c_region = 2;
                    this.$nextTick(() => {
                      //先删一遍class
                      this.removeClass(this.$refs.province_btn, "choosedProvince");
                      this.removeClass(this.$refs.city_btn, "choosedProvince");
                      this.removeClass(this.$refs.area_btn, "choosedProvince");
                      //设置省
                      this.$refs.province_btn[i].classList.add("choosedProvince");
                      //设置市
                      this.$refs.city_btn[drawC].classList.add("choosedProvince");
                      //设置区
                      this.$refs.area_btn[c].classList.add("choosedProvince");
                      this.c_value = `${provincename.name}-${this.citys[drawC].name}-${areaArr[c].name}`;
                    })
                    break;
                  }
                }
              }
            }
          }
        } else {
          this.clear();
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
        this.cityChooseArr = [];
        this.areaChoose = {};
        this.c_value = ''; //清空文字
        this.c_region = 0; //面板位置恢复
        this.rangeSet = 0; //无区域差别选择恢复为0
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
      clearReturn() {
        this.clear();
        let region = {
          province: {
            code: '',
            name: ''
          },
          city: {
            code: '',
            name: ''
          },
          area: {
            code: '',
            name: ''
          }
        }
        this.$emit("cRegion", region);
      },
      //赋值选择的地区并返回出数据
      returnRegion() {
        let region;
        if (((!this.provinceGet) && (!this.cityGet) || (this.fullrange && this.rangeSet == 3)) && this.isEmptyObj(this.provinceChoose) && this.isEmptyObj(this.cityChoose) && this.isEmptyObj(this.areaChoose)) { //如果省、市都没有设置 或者 无区域差别情况下三个区域都已经有值 //三个区域必须有值
          region = {
            province: this.provinceChoose,
            city: this.cityChoose,
            area: this.areaChoose
          }
          //给输入框赋值
          this.c_value = `${this.provinceChoose.name}-${this.cityChoose.name}-${this.areaChoose.name}`;
        } else if ((!this.provinceGet) && this.cityGet || (this.fullrange && this.rangeSet == 2)) { //如果设置了市并且没有设置省 或者 无区域差别情况下省市区域都已经有值
          region = {
            province: this.provinceChoose,
            city: this.cityChoose
          }
          //给输入框赋值
          this.c_value = `${this.provinceChoose.name}-${this.cityChoose.name}`;
        } else if (this.provinceGet || (this.fullrange && this.rangeSet == 1)) { //如果设置了省 或者 无区域差别情况下省区域都已经有值
          region = {
            province: this.provinceChoose
          }
          //给输入框赋值
          this.c_value = this.provinceChoose.name;
        }

        //将数据返回出去
        this.$emit("cRegion", region);
      },
      //多选城市返回数据
      multpleReturn() {
        if (this.cityChooseArr.length == 0) return; //如果没有选择市
        let region = {
          province: this.provinceChoose,
          city: this.cityChooseArr
        }
        //给输入框赋值
        let cityName = '';
        this.cityChooseArr.forEach((city, index) => {
          if (index != this.cityChooseArr.length - 1) {
            cityName += `${city.name}, `;
          } else {
            cityName += city.name;
          }
        })

        this.c_value = `${this.provinceChoose.name} (${cityName})`;
        //关闭选择框
        this.region_select = false;
        this.$emit("cRegion", region);
      },
      //返回单个值
      returnSingle(){ //fullrange
        //如果省都没有选的,return
        if(!this.isEmptyObj(this.provinceChoose)) return;
        //判断选到了哪个区域
        if(this.isEmptyObj(this.provinceChoose) && this.isEmptyObj(this.cityChoose) && this.isEmptyObj(this.areaChoose)){
          this.rangeSet = 3;
          //返回数据
          this.returnRegion();
          //关闭选择框
          this.region_select = false;
          return;
        }
        if(this.isEmptyObj(this.provinceChoose) && this.isEmptyObj(this.cityChoose)){
          this.rangeSet = 2;
          //返回数据
          this.returnRegion();
          //关闭选择框
          this.region_select = false;
          return;
        }
        if(this.isEmptyObj(this.provinceChoose)){
          this.rangeSet = 1;
          //返回数据
          this.returnRegion();
          //关闭选择框
          this.region_select = false;
          return;
        }
        
      },
      //判断对象是否为空
      isEmptyObj(obj){
        let result = null;
        Object.keys(obj).length > 0 ? result = true : result = false;
        return result
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
            this.provinceIndex = i; //省所在的索引
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
        const code2 = code.substr(0, 2); //通过这个找省
        const code4 = code.substr(0, 4); //通过这个找市
        let provinceIndex;
        for (let i = 0; i < this.province.length; i++) {
          if (this.province[i].code.substr(0, 2) == code2) {
            this.provinceChoose = this.province[i]; //设置选中的省
            this.removeClass(this.$refs.province_btn, "choosedProvince");
            this.$refs.province_btn[i].classList.add("choosedProvince");
            provinceIndex = i; //记录哪个省
            this.provinceIndex = i;
            //知道了是哪个省之后就可以拿市了
            this.citys = this.domain[i].city;
            this.$nextTick(() => {
              for (let i = 0; i < this.citys.length; i++) {
                if (this.citys[i].code.substr(0, 4) == code4) {
                  this.cityChoose = this.citys[i]; //设置选中的市
                  this.removeClass(this.$refs.city_btn, "choosedProvince");
                  this.$refs.city_btn[i].classList.add("choosedProvince");

                  if (this.cityGet) { //如果设置了市，直接返回
                    if (!this.multipleCity && !this.fullrange) {
                      //关闭选择框
                      this.region_select = false;
                      this.returnRegion();
                    }
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
           //需要更新省index
            this.provinceIndex = i;
            this.removeClass(this.$refs.province_btn, "choosedProvince");
            this.$refs.province_btn[i].classList.add("choosedProvince");
            provinceIndex = i; //记录哪个省
            this.provinceIndex = i;
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
                      if(!this.fullrange){
                         this.returnRegion();
                         //关闭选择框
                        this.region_select = false;
                      }
                      
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
    color: #fff !important;
  }

  .xz_selector i {
    color: #424a5e;
    font-size: 14px;
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
    padding: 0 28px 0 15px;
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
    position: relative;
  }

  .multpleReturn {
    position: absolute;
    font-size: 14px;
    color: #fff;
    border-radius: 5px;
    width: 100px;
    height: 80%;
    top: 10%;
    background: #5f4b8b;
    right: 2%;
    outline: none;
    border: none;
    cursor: pointer;
    z-index: 10;
  }

  .clearChoose {
    position: absolute;
    font-size: 14px;
    color: #fff;
    border-radius: 5px;
    width: 100px;
    height: 80%;
    top: 10%;
    background: #c82a2e;
    right: 135px;
    outline: none;
    border: none;
    cursor: pointer;
    z-index: 10;
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
    color: #fff !important;
    background: #5f4b8b;
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

  .xz_checkbox input[type="checkbox"]:checked+.xz_custom_checkbox::before {
    content: '';
    position: absolute;
    left: 4px;
    top: 5px;
    background: #424a5e;
    width: 2px;
    height: 7px;
    border-radius: 10px;
    transform: rotateZ(-40deg);
  }

  .xz_checkbox input[type="checkbox"]:checked+.xz_custom_checkbox::after {
    content: '';
    position: absolute;
    right: 3px;
    top: 1px;
    background: #424a5e;
    width: 2px;
    height: 10px;
    transform: rotateZ(26deg);
    border-radius: 10px;
  }
</style>