<template>
  <div class="xz_selecter">
    <div class="xz_domain">
      <div class="xz_search_input">
        <input type="text" placeholder="搜索">
      </div>
      <div class="xz_total_domain">
        <!--省选择-->
        <div class="xz_province">
          <ul>
            <!-- <li class="choosedProvince" ref="province_btn" @click="chooseProvince(-1)">热门城市</li> -->
            <li v-for="(p,index) in domain" :key="p.province.code" @click="chooseProvince(index,p)" ref="province_btn">
              {{p.province.name}}
            </li>
          </ul>
        </div>
        <!--市-->
        <div class="xz_city">
          <span class="single_city" v-for="city in citys" :key="city.code">{{city.name}}</span>
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
      }
    },
    data() {
      return {
        province: {}, //省
        city: [], //市
        area: [], //区
        domain: [{
          area: [],
          city: [],
          province: {
            name: '热门城市',
            code: '000000'
          }
        }], //整理出来的分类集合 (按省分)
        //测试的热门城市
        testHot: [{
          name: '北京',
          code: '111111'
        }, {
          name: '杭州',
          code: '222222'
        }, {
          name: '深圳',
          code: '333333'
        }, {
          name: '上海',
          code: '444444'
        }, {
          name: '广州',
          code: '555555'
        }],
        //选择的省的城市
        citys: []
      }
    },
    created() {
      this.province = Data.province;
      this.city = Data.city;
      this.area = Data.area;
      this.count = 0;
      //热门城市 (需要监听，获取到的数据可能会延迟)
      this.domain[0].city = this.citys = this.testHot;

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
      console.log(this.domain);
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
        }
      },
      //移除class
      removeClass(lists, classname) {
        lists.forEach((list) => { //其他项去除class
          if (list.classList.value.indexOf(classname) != -1) list.classList.remove(classname);
        })
      }
    }
  }
</script>
<style scoped>
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

  .xz_province .choosedProvince {
    background: #fff;
    color: #5f4b8b;
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
    width: 740px;
    background: #fff;
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

  .xz_city {
    width: 585px;
    height: 100%;
    overflow-y: scroll;
    padding: 0 10px;
    box-sizing: border-box;
  }

  .xz_city .single_city {
    display: inline-block;
    width: 25%;
    cursor: pointer;
    font-size: 14px;
    color: #424a5e;
    line-height: 25px;
    padding: 10px 0;
  }

  .xz_city .single_city:hover {
    color: #5f4b8b;
  }
</style>