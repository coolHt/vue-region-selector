# region-selector (全屏遮罩层式)
hav full functionality to choose the region u want, but my want is yours, temporarily, lol.

### 2019.7.16 show time 🤟

#### 数据来源 (正规渠道，最新数据，每次更新都获取这里的数据)
http://www.mca.gov.cn/article/sj/xzqh/2019/201901-06/201906211421.html

### 暂定功能
#### 可自定义热门城市
#### 支持城市可多选(省，区县暂不支持) --> 测试中
#### 提供搜索 ✅


### 使用方式 (当前版本2.9.1)
```
npm install region-selector ---save
Vue.use(regionSelector)
<regionSelector></regionSelector>
```

### 2019.8.21 
#### 功能一 (选择省/市/区)

#### 功能二 (设置初始化数据)

### 2019.8.22 
#### 功能三 (搜索省/市/区)
### 2019.8.23 
#### 功能四 (切换选择模式: 省/市(可多选)/区)
####
####
## 自定义属性
###
属性名|数据格式|默认值|说明
---|---|---|---|
selected|Object<br>`{` <br> &nbsp;&nbsp;`  province: '000000'`<br>&nbsp;&nbsp;`  city: '000000'`<br>&nbsp;&nbsp;`  area: '000000'` <br> `}`|{}| 1、初始化地区数据,三个数据必须都有效，否则无法初始化 <br>2、如果初始化数据匹配城市的多选模式,city的属性值需要设置为当前同一省份下的城市code数组(暂不支持跨省多选城市),单个城市值类型可以为String<br>注意: 初始化的数据和选择的模式有关(省/市/区)
search|Boolean|false|调用搜索框
areaGet|Boolean|true|默认选择到区域，如设置了省或市模式，则会被显示为省模式或市模式
cityGet|Boolean|false|选择到城市，如果同时设置了省，则会显示为省模式
provinceGet|Boolean|false|选择到省份
multpleCity|Boolean|false|城市多选，如果需要使用此属性,必须启用市模式(设置cityGet = true)，且provinceGet不能为true
####

####
## 自定义方法
###
方法名|返回值|说明
---|---|---|
cRegion|`{` <br> &nbsp;&nbsp;`  province: {code: '000000', 'name': '省'}`<br>&nbsp;&nbsp;`  city: {code: '000000', 'name': '市'}`<br>&nbsp;&nbsp;`  area: {code: '000000', 'name': '区'}` <br>`}`|返回完整的省份数据，城市数据和区域数据, 在城市的多选模式下，city属性返回的是city数组



