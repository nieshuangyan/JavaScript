# echarts使用总结

## 问题

### 渲染echarts的DOM外层需要包一层父div

vue中如果if-else类似的渲染逻辑，需要在渲染echarts的DOM的外层包一层父div，不然当调用echarts的api：clear()方法后，会出错

vue出错代码

```vue
<div v-if="isEmpty">{{emptyContent}}<div>
<div v-else-if="isCarouse"><div>
<div v-else-if="isForm"><div>
<div v-else ref=""chartBox><div>  <!-- echarts渲染在该div上面 -->
```
