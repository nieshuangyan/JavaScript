# echarts使用总结

## 问题

### 渲染echarts的DOM外层需要包一层父div

vue中如果if-else类似的渲染逻辑，需要在渲染echarts的DOM的外层包一层父div，不然当调用echarts的api：clear()方法后，会出错

vue出错代码

```vue
<!-- vue -->
<div v-if="isEmpty" class="empty">{{emptyContent}}<div>
<div v-else-if="isCarouse" class="carouse"><div>
<div v-else-if="isForm" class="form"><div>
<div v-else class="chart chart-box" ref="chartBox"><div>  <!-- echarts渲染在该div上面 -->

<!-- 不满足isEmpty、isCarouse、isForm浏览器html渲染结果 -->
<div data-v-43edeb20="" class="chart chart-box" _echarts_instance_="ec_1606549263583" style="-webkit-tap-highlight-color: transparent; user-select: none; position: relative;">   <div style="position: relative; width: 599px; height: 173px; padding: 0px; margin: 0px; border-width: 0px; cursor: default;">
    <canvas data-zr-dom-id="zr_0" width="599" height="173" style="position: absolute; left: 0px; top: 0px; width: 599px; height: 173px; user-select: none; -webkit-tap-highlight-color: rgba(0, 0, 0, 0); padding: 0px; margin: 0px; border-width: 0px;"></canvas>
  </div>
  <div>/div>
</div>

<!-- 数据交互过程中，突然满足isEmpty的浏览器html渲染结果 -->
<div data-v-43edeb20="" class="empty" _echarts_instance_="ec_1606549263583" style="-webkit-tap-highlight-color: transparent; user-select: none; position: relative;">
  <div style="position: relative; width: 599px; height: 173px; padding: 0px; margin: 0px; border-width: 0px; cursor: default;">
    <canvas data-zr-dom-id="zr_0" width="599" height="173" style="position: absolute; left: 0px; top: 0px; width: 599px; height: 173px; user-select: none; -webkit-tap-highlight-color: rgba(0, 0, 0, 0); padding: 0px; margin: 0px; border-width: 0px;"></canvas>
  </div>
  无供应商相关数据
</div>
```

```javascript
// mounted
this.chart = this.$echarts.init(this.$refs.chartBox)
this.chart.setOption({...})
```

一开始是不满足isEmpty、isCarouse、isForm，class="chart"的div被渲染出来，此时echarts也会被渲染出来
数据交互过程中，突然满足isEmpty，视图切换，就需要执行

```javascript
this.chart.clear()
```
此时，class="empty"的div被错误渲染，该div渲染混乱，之前echarts的画布canvas标签渲染在该div的子元素上，且之前canvas直接父标签的属性强加到该div上

修改后正常渲染，视图切换也正常渲染

```vue
<!-- vue -->
<div v-if="isEmpty" class="empty">{{emptyContent}}<div>
<div v-else-if="isCarouse" class="carouse"><div>
<div v-else-if="isForm" class="form"><div>
<div v-else class="chart">
  <div class="chart-box" ref="chartBox"></div>
<div> 

<!-- 不满足isEmpty、isCarouse、isForm浏览器html渲染结果 -->
<div data-v-43edeb20="" class="inherit chart-box" _echarts_instance_="ec_1606551227626" style="-webkit-tap-highlight-color: transparent; user-select: none; position: relative;">   <div style="position: relative; width: 599px; height: 173px; padding: 0px; margin: 0px; border-width: 0px; cursor: default;">
    <canvas data-zr-dom-id="zr_0" width="599" height="173" style="position: absolute; left: 0px; top: 0px; width: 599px; height: 173px; user-select: none; -webkit-tap-highlight-color: rgba(0, 0, 0, 0); padding: 0px; margin: 0px; border-width: 0px;"></canvas>
  </div>
  <div></div>
</div>

<!-- 数据交互过程中，突然满足isEmpty的浏览器html渲染结果 -->
<div data-v-43edeb20="" class="empty">无供应商相关数据</div>
```
