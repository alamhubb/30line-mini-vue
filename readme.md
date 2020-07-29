梦回2013年，学习尤大vue的第一行代码(vue的0.1版本)，看如何用30行代码实现vue(超简洁，包含核心功能数据绑定，仅作为学习交流，方便理解，适合初学者)

非非非标题党，干货预警！！！

先奉上2013年尤大写的vue的第一行代码github源码地址镇贴： vue 官方仓库 0.1分支explorations/getset.html文件

[尤大0.1版本40行代码实现vue源码地址github地址](https://github.com/vuejs/vue/blob/871ed9126639c9128c18bb2f19e6afd42c0c5ad9/explorations/getset.html)

引读：

本篇文章，用30行代码实现0.1版本vue，是参照尤大2013年，在git上传的vue的第一行代码(vue的0.1)版本代码思路实现的，尤大仅用40行代码就实现了0.1版本的vue。

为了更方便学习和交流，我们基于尤大的代码省略了一些内容，提取核心功能代码，调整了代码顺序，修改了部分变量命名，增加了注释，仅剩30行代码，更加简洁，直观的向大家展示vue的核心功能实现逻辑。

本人也是个小菜鸟，整理本篇文章用了一天的时间，不足之处欢迎大家指正批评，文中代码的具体实现细节存在不严谨的地方，请大家忽略，本文仅用来交流vue的实现思想，并未深究细节。

正文：

本篇文章中代码仅实现了vue的核心功能数据绑定，未实现其他功能。

文件可直接打开复制到html中运行，查看效果

我们先用文字来简单描述下核心代码的实现逻辑

1.获取vue根结点的dom，替换dom中的{{}}模板语法，并根据模板{{msg}}中使用data中的属性名称(msg)为其打上标志，方便之后寻找哪些dom使用了模板语法
2.记录模板语法中使用了data中的哪些属性，方便在这些值变化时更新dom
3.遍历这些属性，并根属性名称找到对应的dom，移除标志(标志用来寻找dom，此时已经获得了dom，所以移除标志)
4.拦截属性的set、get方法，在set属性时，更新dom的textContent实现数据变化更新视图的功能

接下来看代码如下：

```javascript
//实例化vue
const app = new vue('app', {msg: 'hello'})

//id: app
//initData: {msg: "hello"}
function vue(id, initData) {
  //vue中一般都包含el属性，代表vue实例对应的dom元素
  const vueDom = this.el = document.getElementById(id) //获取 #app dom <div id="app">...</div>
  //vue的data属性，尾部为data赋值
  const data = this.data = {}

  //定义常量，用来标示需要数据绑定的dom元素使用
  const bindingMark = 'bind-dom-flag'
  //定义临时变量来存储模板中使用了data中的值的列表
  const dataAttrs = {}

  //替换#app dom中的模板内容
  vueDom.innerHTML = vueDom.innerHTML.replace(/{{(.*)}}/g, (match, dataAttrName) => {
    //用来记录模板中使用了data中的哪些值，本案例模板中只使用data中的msg
    dataAttrs[dataAttrName] = {} // {msg: {}}
    //给使用了模板的dom打上标志，bindingMark = data中的值msg，以便将这些dom放进 dataAttrs 中
    return '<span ' + bindingMark + '="' + dataAttrName + '"></span>'
  })

  //遍历dom中使用的data属性
  for (const dataAttrName in dataAttrs) {
    //获取data对应的dom列表
    const doms = vueDom.querySelectorAll('[' + bindingMark + '="' + dataAttrName + '"]')
    //移除dom上的标志，标志用来获取dom列表，获取后就可以将标志删除了
    doms.forEach((e) => {
      e.removeAttribute(bindingMark)
    })
    //获取定义临时变量中的 dataAttr 对象，defineProperty实现数据绑定需要借助这个对象来实现
    const dataAttrObj = dataAttrs[dataAttrName]
    //劫持data属性的set，get方法
    Object.defineProperty(data, dataAttrName, {
      get() {
        //返回临时变量dataAttr中的attr
        return dataAttrObj.val
      },
      set(newVal) {
        //更新data中此属性对应的所有dom，更新所有模板中使用了msg的dom
        doms.forEach((dom) => {
          dataAttrObj.val = dom.textContent = newVal
        })
      }
    })
  }

  //将外部传入的初始化值赋值给vue实例的data属性
  for (const dataAttrName in initData) {
    //赋值，触发set，更新dom
    this.data[dataAttrName] = initData[dataAttrName]
  }
}
```

html如下：

```html
<div id="app">
  <p>{{msg}}</p>
  <p>{{msg}}</p>
  <p>{{msg}}</p>
</div>
```
