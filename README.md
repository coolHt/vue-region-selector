# region-selector (全屏遮罩层式)
Have full functionality to choose the region u want, but my want is yours, temporarily, lol.
### 各位哥哥姐姐觉得使用方便的话给个star吧。⭐️祝各位靓仔一天更比一天帅，靓女一天更比一天美
### 🤟🌟🥴🤠🤸🤺🤾🧟‍♂️🧛🏿‍♂️🧙‍♂️👨‍🎤🌳🏅

#### 数据来源 (正规渠道，最新数据，每次更新都获取这里的数据)
http://www.mca.gov.cn/article/sj/xzqh/2019/201901-06/201906211421.html

### 暂定功能
#### 可自定义热门城市 ❌
#### 支持城市可多选(省，区县暂不支持) ✅
#### 提供搜索 ✅


### 使用方式 (当前版本2.9.3)
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
### 2019.8.25 
#### 功能四 (切换选择模式: 省/市(可多选)/区)
####
####
## 自定义属性
###
属性名|数据格式|默认值|说明
---|---|---|---|
selected|String 或 Array(表示多选，仅在城市选择的多选模式下才能完全匹配，否则默认匹配第一个值)|''| 1、初始化的值与当前的选择模式相匹配(比如初始值为城市的code在选择模式为区的情况下无效) <br> 2、可以为城市选择的多选模式提供一个城市code 数组，如果城市列表中不包含数组中的值会被自动过滤掉
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



