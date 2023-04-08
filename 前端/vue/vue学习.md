
* 监听某值是否变化

```ts
watch(() => scrollTop.value, (newVal, oldVal) => {
  console.log('#######newVal=>', newVal)
  console.log('#######oldVal=>', oldVal)
})
```

* 鼠标滚到事件

```ts
const onScroll = () => {
  let scrollTop = document.documentElement.scrollTop;
  for (let i = 0; i < sectionPosition.length; i++) {
    if (sectionPosition[i]["offsetTop"] < scrollTop) {
      currentMenu.value = sectionPosition[i]["idName"]
      console.log('############currentMenu.value=>', currentMenu.value)
    }
  }

}

onMounted(() => {
  // getSectionPosition()
  window.addEventListener("scroll", onScroll);
});

// HTML精确定位:scrollLeft,scrollWidth,clientWidth,offsetWidth   
// scrollHeight: 获取对象的滚动高度。   
// scrollLeft:设置或获取位于对象左边界和窗口中目前可见内容的最左端之间的距离   
// scrollTop:设置或获取位于对象最顶端和窗口中可见内容的最顶端之间的距离   
// scrollWidth:获取对象的滚动宽度   
// offsetHeight:获取对象相对于版面或由父坐标 offsetParent 属性指定的父坐标的高度   
// offsetLeft:获取对象相对于版面或由 offsetParent 属性指定的父坐标的计算左侧位置   
// offsetTop:获取对象相对于版面或由 offsetTop 属性指定的父坐标的计算顶端位置   
// event.clientX 相对文档的水平座标   
// event.clientY 相对文档的垂直座标   
// event.offsetX 相对容器的水平坐标   
// event.offsetY 相对容器的垂直坐标   
// document.documentElement.scrollTop 垂直方向滚动的值   
// event.clientX+document.documentElement.scrollTop 相对文档的水平座标+垂直方向滚动的量

// 参考: https://blog.csdn.net/weixin_49295874/article/details/123152922
```

* 在Vue组件中，我们使用@wheel事件来检测滚轮事件
```vue
<template>
  <div class="container" @wheel="onWheel">
    <div class="layer1"></div>
    <div class="layer2"></div>
  </div>
</template>

<script>
export default {
  methods: {
    onWheel(event) {
      // 获取鼠标滚轮滚动的距离
      const deltaY = event.deltaY;

      // 计算第二个图层需要移动的距离
      const distance = deltaY * 0.5;

      // 使用transform属性移动第二个图层
      const layer2 = document.querySelector('.layer2');
      layer2.style.transform = `translateY(-${distance}px)`;
    }
  }
}
</script>

<style>
.container {
  position: relative;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

.layer1 {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #f0f0f0;
}

.layer2 {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #ccc;
  transition: transform 0.3s ease-out;
}
</style>

```


参考:  
[基于vue+vant搭建H5通用架子](https://juejin.im/post/5db52dedf265da4d495c3fb8)  
[HTML5中盒子模型重点总结](https://blog.csdn.net/baidu_29343517/article/details/81988791)


[animate动画库](https://daneden.github.io/animate.css/)

[animation实现一闪一闪效果](https://blog.csdn.net/caiyongshengcsdn/article/details/79397474)

[在vue中使用animate.css](https://www.cnblogs.com/gluncle/p/9662410.html)

[css实现心跳效果](https://blog.csdn.net/weixin_39147099/article/details/85018205)


[CSS实现盒子模型水平居中、垂直居中、水平垂直居中的多种方法](https://blog.csdn.net/Li_dengke/article/details/81193451)


[vite配置https](https://juejin.cn/post/7134582855006421000)