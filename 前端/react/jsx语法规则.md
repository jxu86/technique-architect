```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport"
        content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>jsx语法规则</title>
  <style>
    .title {
      background-color: bisque;
      width: 200px;
    }
  </style>
</head>
<body>
<!-- 容器 -->
<div id="test"></div>
<!-- 引入react核心库 -->
<script src="../js/react.development.js"></script>
<!-- 引入react-dom 用于支持react操作DOM -->
<script src="../js/react-dom.development.js"></script>
<!-- 引入babel用于jsx转js -->
<script src="../js/babel.min.js"></script>

<script type="text/babel">/* babel转义 */
const myId = 'aTgUiGu';
const myData = 'Hello,React';
const spanStyle = {
  color: 'green',
  fontSize: '30px'
};
const Fragment = React.Fragment;
// 1. 创建虚拟DOM
const VDOM = (
  <Fragment>
    <h2 id={myId.toLowerCase()} className="title">
      <span style={spanStyle}>{myData.toLowerCase()}</span>
    </h2>
    <input type="text"/>
  </Fragment>
)
// 2. 渲染虚拟DOM到页面
// render(虚拟DOM,容器)
ReactDOM.render(VDOM, document.getElementById('test'));
/*
* jsx语法规则：
* 1. 定义虚拟dom，不要使用引号
* 2. 标签中混入js表达式，使用插值表达式{}
* 3. 样式的类名指定不要使用class,使用className
* 4. 内联样式使用style={{key:value}}形式实现
* 5. 虚拟DOM必须只有一个根标签
* 6. 标签必须闭合
* 7. 标签首字母
*    （1）小写字母开头，则将该标签转为html同名元素，若html无该标签对应的同名元素，报错
*    （2）若大写字母开头，react就去渲染对应的组件，若无组件，则报错
* */
</script>
</body>
</html>
```