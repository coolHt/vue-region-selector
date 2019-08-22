# region-selector (全屏遮罩层式)
hav full functionality to choose the region u want, but my want is yours, temporarily, lol.

### 2019.7.16 show time 🤟

#### 数据来源 (正规渠道，最新数据，每次更新都获取这里的数据)
http://www.mca.gov.cn/article/sj/xzqh/2019/201901-06/201906211421.html

### 暂定功能
#### 可自定义热门城市
#### 支持城市可多选(省，区县暂不支持)
#### 提供搜索


### 2019.8.21 
#### 功能一 (选择省/市/区)
####
####
## 自定义属性
###
属性名|数据格式|说明
---|---|---|
selected|`{` <br> &nbsp;&nbsp;`  province: '000000'`<br>&nbsp;&nbsp;`  city: '000000'`<br>&nbsp;&nbsp;`  area: '000000'` <br> `}`|初始化地区数据,三个数据必须都有效，否则无法初始化
####

####
## 自定义方法
###
方法名|返回值|说明
---|---|---|
cRegion|`{` <br> &nbsp;&nbsp;`  province: {code: '000000', 'name': '省'}`<br>&nbsp;&nbsp;`  city: {code: '000000', 'name': '市'}`<br>&nbsp;&nbsp;`  area: {code: '000000', 'name': '区'}` <br>`}`|返回完整的省份数据，城市数据和区域数据



