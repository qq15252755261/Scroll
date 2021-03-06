# Scroll 无滚动条的滚动插件，模拟原生app滚动，支持PC端和手机端，插件可配置部分功能并提供了一些功能性的api
## 查看演示：
如果你想在PC端查看，请打开/src/index.html，通过开发者工具调成手机模式能获得更好的观感体验<br/>
如果你想在手机端查看，请先安装node.js<br/>
接着在cmd命令行中进入Scroll文件夹的src目录并运行如下命令<br/>
### view in mobile
```
node ../node_modules/http-serve/bin/http-serve
```
请保持你的手机和电脑在同一个网络下，你可以通过输出的网址访问它（这是我的网址，每个人的不一样）
```
Starting up http-serve for ./
Available on:
  http://192.168.0.102:8080
  http://127.0.0.1:8080
Hit CTRL-C to stop the server
```
### Scroll插件位于
```
/src/js/Scroll.js
```
### 在自己的项目中引入Scroll插件
```
import Scroll from 'Scroll.js'
```
## 基本使用方法：
```javascript
<html>
  <div id="scroll-wrapper">
    <div id="scroll" class="scroll">
      <!-- scroll content -->
    </div>
  </div>
</html>

<script>
  const scroll_wrapper = document.getElementById('scroll-wrapper')

  new Scroll(scroll_wrapper)
  /*
    或者
    new Scroll('#scroll-wrapper')
    或者
    new Scroll('.scroll-wrapper')
    因为Scroll类既支持传入元素对象，也支持传入id或者class
  */
<script>
```
## 使用教程、注意事项以及插件特点：
### 1. Scroll插件的第一个参数
实际滚动的元素是传入Scroll类的参数的第一个子元素。按上面的例子来说，需要滚动的是id为'scroll'的元素，那么Scroll类中传参的是'scroll'的父元素'scroll_wrapper'。下面我们把“需要滚动的元素”简称为“滚动元素”，它的父元素简称为“父元素”。
### 2. Scroll插件的第二个参数
第二个参数是配置参数，Object类型，还是按上面的例子。<br/><br/>
目前支持三个配置属性：<br/><br/>
noInertia（Boolean：手指滑动时没有惯性）<br/><br/>
noSpringback（Boolean：手指拉到尽头释放没有回弹效果）<br/><br/>
endCanMoveDistance（Number：拉到尽头时，允许超过尽头的极限距离，前提是noSpringback必须为false。可以为像素值，如200, 300……；也可为比例值，父元素高度的比例，如0.1, 0.2……，可简写为.1,, .2……）<br/><br/>
如果没有配置参数，默认是：<br/><br/>
noInertia: undefined —— 手指滑动时有惯性<br/><br/>
noSpringback: undefined —— 手指拉到尽头释放有回弹效果<br/><br/>
endCanMoveDistance: undefined —— 拉到尽头时，允许超过尽头的极限距离是：父元素高度的4分之1或100px中较小的一个
```javascript
<script>
  new Scroll('#scroll-wrapper', {
    noInertia: true, // 手指滑动时没有惯性。
    noSpringback: true, // 手指拉到尽头释放没有回弹效果。
    endCanMoveDistance: 200 // 拉到尽头时，允许超过尽头的极限距离是200px，也可以为0.3或.3等，这时极限距离是父元素高度的30%。
  })
<script>
```
### 3. 使用要求
3.1 滚动元素高度必须大于父元素，否则是没有滚动效果的。<br/><br/>
3.2 父元素的'overflow'属性必须为'hidden'，否则会报错'The current wrapperNode's overflow is "……", it should be "hidden'。
### 4. 随元素节点自动更新
如果一开始滚动元素的高度小于父元素自然是无法滚动的，后来滚动元素的内容变更了（比如进行了ajax请求），现在它的高度大于父元素，那么这时页面是可以滚动的，无需手动更新。
### 5. api
Scroll类的实例提供了一些功能性的api方法<br/><br/><br/>
5.1 `on`<br/><br/><br/>
on方法用来监听滚动过程中的一些事件和特殊动作：<br/><br/>
`'scroll'`：滚动事件。当发生滚动时，调用传入的事件方法，方法的第一个参数是当前滚动的坐标对象，对象的y属性是当前的Y轴移动值(负数)
```javascript
let scroll = new Scroll('#scroll-wrapper')

scroll.on('scroll', function (pos) {
  // pos.y……
})
```
`'pullDown'`：顶部下拉事件。下拉事件有两个钩子函数和一个回调函数<br/><br/>
仅当on方法的第二个参数是对象Object时，才能使用两个钩子函数，它们以对象属性的形式书写。<br/><br/>
enterHook是：滚动页面已经处于顶部，仍然继续下拉的一瞬间触发的钩子函数<br/><br/>
leaveHook是：滚动页面在下拉超过顶部后重新回到顶部的一瞬间 或 手指释放的一瞬间 触发的钩子函数<br/><br/>
它们的回调方法的第一个参数是当前滚动的坐标对象<br/><br/>
callback是：下拉超过顶部后释放时触发的回调。如果回调是同步函数，需要在结束时调用success；如果是异步函数，需要在成功时调用success，失败时调用fail。
```javascript
scroll.on('pullDown', {
  //刚开始下拉时(pos.y大于0)，只触发一次
  enterHook: function (pos) {
    // todo……
  },
  //刚离开下拉时(pos.y小于0或下拉被释放)，只触发一次
  leaveHook: function (pos) {
    // todo……
  },
  //下拉释放时
  callback: function (success, fail) {
    // todo……
    success()
  }
})
```
当on方法的第二个参数是函数Function时，没有enterHook和leaveHook两个钩子，参数Function直接作为释放时的callback<br/><br/>
这里示范了如果回调是异步函数
```javascript
scroll.on('pullDown', function (success, fail) {
  //模拟ajax请求
  ajax.get('/api/json')
  .then(() => {
    // todo……
    success()
  })
  .catch(() => {
    // todo……
    fail()
  })
})
```
`'pullUp'`：底部上拉事件。方法同上。<br/><br/>
使用场景：<br/><br/>
利用监听scroll，可以做出类似外卖软件中左侧商品和右侧滚动页面联动的功能<br/><br/>
利用监听特殊动作，可以用来做下拉刷新和上拉加载的功能等等<br/><br/><br/>
5.2 scrollTo方法<br/><br/><br/>
scrollTo方法可以让滚动页面在指定时间内滚动到指定位置，它接受两个参数：<br/><br/>
第一个参数既可以是元素Element，也可以是数字Number或字符串String。如果是元素，则滚动到那个元素所在的位置；如果是数字(例如200)，则滚动到距离顶部200px的位置；如果是'200px'之类的字符串也可以。<br/><br/>
第二个参数是滚动的时间，可以是数字Number，也可以是字符串String。如果是纯数字，会自动以毫秒ms为单位；如果是字符串，则会以字符串中的单位为时间单位(如'0.3s'，单位就是秒s)；如果字符串中只有数字，依然自动以毫秒ms为单位。
```
//随意假设一个滚动元素中的元素，同时假设它在滚动元素中距离顶部200px
const content = document.getElementById('content')

//我们想某个按钮点击后能在0.4秒内滚动到content元素所在的位置
scroll.scrollTo(content, 400)
/*
或者还可以这样写：
scroll.scrollTo(200, 400)
scroll.scrollTo('200px', 400)
scroll.scrollTo(content, '0.4s')
scroll.scrollTo(content, '400ms')
scroll.scrollTo(content, '400')
*/
```
使用场景：<br/><br/>
利用这个api可以做出类似外卖软件中左侧商品和右侧滚动页面联动的效果<br/><br/>
也可以做一些前往锚点，返回顶部，前往底部的按钮<br/><br/><br/>
5.3 refresh方法<br/><br/><br/>
手动触发更新，不过大部分情况不需要用到
```
scroll.refresh()
```