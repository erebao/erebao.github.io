---
title: 2019年vue-vuejs从入门到精通
date: 2021-02-20 14:11:57
categories:
  - vue2.x
  - vuejs从入门到精通
tags:
  - vue2.x
  - vuejs从入门到精通
---

2019年vue-vuejs从入门到精通
<!--more-->


#### let
ES5之前因为if和for都没有块级作用域的概念，所以在很多时候，我们都必须借助于function的作用域来解决应用外面变量的问题
ES6中，加入了let，let它是有if和for的块级作用域


#### const
注意一：一旦使用const的修饰标识符被赋值之后不能修改
注意二：在使用const定义标识符，必须进行赋值


#### 属性的增强写法
```
const name = 'why'
const age = 18
const height = 1.88

const obj = {
    name,
    age,
    height,
}
```

#### 函数的增强写法
```
const obj = {
    run(){

    },
    eat(){

    }
}
```

#### v-for
基于一个数组来渲染一个列表
```
<div id="app">
    <ul>
        <li v-for="item in movies">{{item}}</li>
    </ul>
</div>
```

#### v-on
绑定事件监听器。事件类型由参数指定。表达式可以是一个方法的名字或一个内联语句
```
<button v-on:click="add">+</button>
<!-- 语法糖写法 -->
<button @click="sub">-</button>
```

.stop(阻止事件冒泡)修饰符的使用
```
<div @click="divClick">
        aaaa
        <button @click.stop="btnClick">按钮</button>
</div>
```

.prevent(阻止事件的默认行为)修饰符的使用
```
<div action="baidu">
        <input type="submit" value="提交" @click.prevent="submitClick">
</div>
```

监听键盘回车按钮
```
<input type="text" @keyup.enter="keyUp">
```

.once(只调用一次)修饰符的使用
```
<button @click.once="btn2Click">按钮2</button>
```

#### v-once
只渲染元素和组件一次
```
<h2 v-once>{{message}}</h2>
```


#### v-html
更新元素的 innerHTML
```
<h2 v-html="url"> </h2>
```

```
......
	url: '<a href="http://www.baidu.com">百度一下</a>'
......
```

#### v-text
更新元素的 textContent
```
<span v-text="msg"></span>
<!-- 和下面的一样 -->
<span>{{msg}}</span>
```

#### v-pre
跳过编译
```
<h2 v-pre>{{message}}</h2>
```

#### v-cloak
在vue解析之前，div中有一个属性v-cloak
在vue解析之后，div中没有一个属性v-cloak

解析完成之后才显示：
```
<style>
        [v-cloak]{
            display: none;
        }
</style>
```
```
<div id="app" v-cloak>
    {{message}}
</div>
```

#### v-bind
动态绑定属性
```
<a v-bind:href="aHref">百度一下</a>

<!-- 语法糖的写法 -->
<a :href="aHref">百度一下</a>
```

#### computed
计算属性
通过定义methods 不会被缓存
通过computed 会被缓存


#### v-if v-else
条件判断
```
<div id="app">
  <h2 v-if="score >= 90">优秀</h2>
  <h2 v-else-if="score >= 80">良好</h2>
  <h2 v-else-if="score >= 60">及格</h2>
  <h2 v-else>不及格</h2>
</div>
```

#### v-show
当条件为false时，v-show只是给我们的元素添加了一个行内样式：
```
display:none
```

#### 数组操作
push
```
this.letters.push('aaa')
this.letters.push('aaaa', 'bbbb', 'cccc', 'dddd')
```

pop删除数组中的最后一个元素
```
this.letters.pop()
```

shift删除数组中的第一个元素
```
this.letters.shift()
```

unshift在数组最前面添加元素
```
this.letters.unshift('aaa')
this.letters.unshift('aaaa', 'bbbb', 'cccc', 'dddd')
```

splice删除/插入/替换元素
```
// 删除元素：第二个元素传入你要删除几个元素（如果没有传，就删除后面所有的元素）
// 替换元素：第二个参数，表示我们要替换几个元素，后面是用于替换前面的元素
// 插入元素：第二个参数，传入0，并且后面跟上要插入的元素
this.letters.splice(1, 3, 'm', 'n', 'l', 'x')
this.letters.splice(1, 0, 'x', 'y', 'z')
```

sort排序
```
this.letters.sort()
```

reverse反转
```
this.letters.reverse();
```

通过索引值修改数组中的元素
```
//this.letters[0]='bbbbbb'  // 错误写法
this.letters.splice(0, 1, 'bbbbbb')
// set(要修改的对象, 索引值, 修改后的值)
Vue.set(this.letters, 0, 'bbbbbb')
```

#### v-model
双向绑定
```
<div id="app">
    <input type="text" v-model="message">
</div>
```

lazy 取代 input 监听 change 事件
```
<input type="text" v-model.lazy="message">
```

number 输入字符串转为有效的数字
```
<input type="number" v-model.number="age">
```

trim 输入首尾空格过滤
```
<input type="text" v-model.trim="name">
```

#### 组件化
1.创建组件构造器对象
```
Vue.extend({
        template: ''
})
```
2.注册组件
```
Vue.component('my-cpn', cpnC)
```
3.使用组件
```
<div id="app">
    <cpn></cpn>
    <cpn></cpn>
    <cpn></cpn>
</div>
```

注册局部组件
```
const app = new Vue({
        el: '#app',
        components: {
            // cpn使用组件时的标签名
            cpn: cpnC
        }
})
```

#### 组件分离写法
```
<!-- 1.script标签，类型要是text/x-template -->
<!--<script type="text/x-template" id="cpn">
    <div>
        <h2>我是标题script</h2>
        <p>我是内容，hhhhhhhhhhhhh</p>
    </div>
</script>-->

<!-- 2.template标签 -->
<template id="cpn">
    <div>
        <h2>我是标题template</h2>
        <p>我是内容，hhhhhhhhhhhhh</p>
    </div>
</template>
```

```
Vue.component('cpn', {
        template: `#cpn`
})
```

#### 组件中的data
必须是函数
```
Vue.component('cpn',{
        template: '#cpn',
        data(){
            return{
                counter: 0
            }
        }
})
```

#### 组件传递数据
父传子：props
子传父: this.$emit()

#### watch
侦听器，用来观察和响应 Vue 实例上的数据变动

#### 父组件访问子组件
```
// 1. $children方式
/*console.log(this.$children);
for (let c of this.$children){
	console.log(c.name)
	c.showMessage()
}*/
// 2. $refs
console.log(this.$refs.aaa.name);
```

#### 子组件访问父组件
```
// 1. 访问父组件$parent
console.log(this.$parent);
console.log(this.$parent.name);
// 2. 访问根组件$root
console.log(this.$root);
console.log(this.$root.message);
```

#### 插槽

具名插槽
```
<slot name="left"></slot>
```

作用域插槽
```
<!-- 目的是获取子组件中的pLanguages -->
<template slot-scope="slot">
	<!--<span v-for="item in slot.data">{{item}} - </span>-->
	<span>{{slot.data.join(' - ')}}</span>
</template>

<script>
    const app = new Vue({
        el: '#app',
        data: {
            message: '你好啊',
        },
        components: {
            cpn: {
                template: '#cpn',
                data() {
                    return {
                        pLanguages: ['JavaScript', 'C++', 'Java', 'C#', 'Python', 'Go', 'Swift']
                    }
                }
            }
        }
    })
</script>
```
