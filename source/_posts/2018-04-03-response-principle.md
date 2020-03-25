---
title: 依赖收集和响应式原理
date: 2018-04-03 10:34:03
tags: [VUE,JS]
categories: [VUE]
---
博主最近在学习Vue框架，对其中的响应式原理感觉非常有趣，于是上网查了各种资料，总结了这篇文章，希望对大家有帮助

## 一、Vue的响应式原理
其实大家上网翻阅资料也知道Vue的响应式原理其实是通过数据劫持来达到监听数据的目的。说到数据劫持就不能不提到[Object.defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)方法了。
这个方法的具体作用就是使属性变得可观测，属性变得可观测以后，如何反映到视图上，就是我们需要研究的内容了。

## 二、监听属性
首先，我们定义一个数据对象：
```bash
const person = {
  sex: '女',
  age: 30
}
```
我们定义了一个30岁的女人。现在我们可以通过person.sex和person.age读取属性。但是当这个数据被修改时，我们却不知道。这时候就要用到Object.defineProperty方法了。那到底应该如何做呢？看下面代码
```bash
let person = {}
let val = 60
Object.defineProperty(person, 'age', {
  get () {
    console.log('age is getting')
	  return val
  },
  set (newVal) {
    console.log('age is setting')
    val = newVal
  }
})
```
我们通过Object.defineProperty方法，给person定义了一个age属性，这个属性在被读写时候都会被告知了，我们可以在控制台试一下。

{% asset_img 1.png %}

可以看到我们已经可以监听这个人的年龄属性了。为了可以监听这个人的所有属性，我们可以写一个方法

```bash
function ObserveProperty (obj, key, val) {
  Object.defineProperty(obj, key, {
    get () {
      // 触发getter
      console.log(`${key} getting`)
      return val
    },
    set (newVal) {
      // 触发setter
      console.log(`${key} setting`)
      val = newVal
    }
  })
}

function ObserveObj (obj) {
  const keys = Object.keys(obj)
  keys.forEach((key) => {
    ObserveProperty(obj, key, obj[key])
  })
  return obj
}

let person = ObserveObj({
  sex: '女',
  age: 30
})
```

现在我们就可以知道这个30岁女人的一切情况了(奸笑。。。。)

## 三、计算属性
现在我们已经可以监听这个女人了，但是我们仍不满足，我们还需要她在修改属性的时候，能够告诉我们其他信息。假设可以这样：
```bash
watcher(person, 'type', () => {
  return person.age > 18 ? '御姐' : '萝莉'
})
```
我们定义了一个watcher作为"监听器",他监听了person的type属性。而person.type的值取决于person.age,我们就可以说person.age是person.type的依赖，我们称person.type为"计算属性"。

现在我们可以尝试着构造这个监听器。
```bash
function onComputedUpdate (val) {
	console.log(`我的类型是: ${val}`)
}

function watcher (obj, key, fn) {
  Object.defineProperty(obj, key, {
    get () {
      const val = fn()
      onComputedUpdate(val)
      return val
    },
    set () {
      console.error('计算属性无法被赋值!')
    }
  })
}

watcher(person, 'type', () => {
  return person.age > 18 ? '御姐' : '萝莉'
})
```

现在我们就在控制台运行看下效果：

{% asset_img 2.png %}

可以看到运行正常，那是不是就成功了呢？骚年，还是太年轻。我们可以看到是我们手动读取的type属性，而不是他主动告诉我们的。我们希望的是当age属性改变时，主动发起通知，那我们到底应该怎么做呢？（啪。。请听下回分解）

## 四、依赖收集

我们知道，当一个可观测对象的属性被读写时，会触发它的getter/setter方法。换个思路，如果我们可以在可观测对象的getter/setter里面，去执行监听器里面的onComputedUpdate方法，是不是就能够实现让对象主动发出通知的功能呢？

由于监听器内的onComputedUpdate()方法需要接收回调函数的值作为参数，而可观测对象内并没有这个回调函数，所以我们需要借助一个第三方来帮助我们把监听器和可观测对象连接起来。

这个第三方就做一件事情——收集监听器内的回调函数的值以及onComputedUpdate()方法。

现在我们把这个第三方命名为“依赖收集器”，一起来看看应该怎么写：

```bash
const Dep = {
	target: null
}
```

就是这么简单，依赖收集器的target就是用来存放监听器的onComputerUpdate()方法的。

接下来我们需要做的就是把onComputedUpdate()方法赋值给Dep.target

```bash
function watcher (obj, key, fn) {
  // 定义一个被动触发函数，当这个“被观测对象”的依赖更新时调用
  const onDepUpdated = () => {
    const val = fn()
    onComputedUpdate(val)
  }

  Object.defineProperty(obj, key, {
    get () {
      Dep.target = onDepUpdated
      // 执行fn()的过程中会用到Dep.target，
      // 当fn()执行完了就重置Dep.target为null
      const val = fn()
      Dep.target = null
      return val
    },
    set () {
      console.error('计算属性无法被赋值！')
    }
  })
}
```

我们在监听器内部定义了一个新的onDepUpdated()方法，这个方法很简单，就是把监听器回调函数的值以及onComputedUpdate()给打包到一块，然后赋值给Dep.target。这一步非常关键，通过这样的操作，依赖收集器就获得了监听器的回调值以及onComputedUpdate()方法。作为全局变量，Dep.target理所当然的能够被可观测对象的getter/setter所使用。

重新看一下我们的watcher实例

```bash
watcher(person, 'type', () => {
  return hero.age > 18 ? '御姐' : '萝莉'
})
```

在它的回调函数中，调用了age属性，触发了对应的getter函数。所以我们需要对ObserveProperty()方法进行改写：

```bash
function ObserveProperty (obj, key, val) {
  const deps = []
  Object.defineProperty(obj, key, {
    get () {
      if (Dep.target && deps.indexOf(Dep.target) === -1) {
        deps.push(Dep.target)
      }
      return val
    },
    set (newVal) {
      val = newVal
      deps.forEach((dep) => {
        dep()
      })
    }
  })
}
```

可以看到，在这个方法里面我们定义了一个空数组deps，当getter被触发的时候，就会往里面添加一个Dep.target。回到关键知识点Dep.target等于监听器的onComputedUpdate()方法，这个时候可观测对象已经和监听器捆绑到一块。任何时候当可观测对象的setter被触发时，就会调用数组中所保存的Dep.target方法，也就是自动触发监听器内部的onComputedUpdate()方法。

至于为什么这里的deps是一个数组而不是一个变量，是因为可能同一个属性会被多个计算属性所依赖，也就是存在多个Dep.target。定义deps为数组，若当前属性的setter被触发，就可以批量调用多个计算属性的onComputedUpdate()方法了。

## 五、尾声

博主的第一篇博客到这里就结束了，虽然有点拾人牙慧，希望大家多多包涵。有问题的话可以提issue，大家一起交流，最后贴上完整代码。

```bash
function ObserveProperty (obj, key, val) {
  const deps = []
  Object.defineProperty(obj, key, {
    get () {
      console.log(`${key} is getting`)
      if(Dep.target && deps.indexOf(Dep.target) === -1) {
        deps.push(Dep.target)
      }
      return val
    },
    set (newVal) {
      console.log(`${key} is setting`)
      val = newVal
      deps.forEach(dep => {
        dep()
      })
    }
  })
}

function ObserveObj (obj) {
  const keys = Object.keys(obj)
  keys.forEach((key) => {
    ObserveProperty(obj, key, obj[key])
  })
  return obj
}

let person = ObserveObj({
  sex: '女',
  age: 30
})
    
const Dep = {
  target: null
}

function onComputedUpdate (val) {
  console.log(`我的类型是: ${val}`)
}

function watcher (obj, key, fn) {
  const onDepUpdated = () => {
    const val = fn()
    onComputedUpdate(val)
  }
  Object.defineProperty(obj, key, {
    get () {
      Dep.target = onDepUpdated
      const val = fn()
      Dep.target = null
      return val
    },
    set () {
      console.log('computed property is not set')
    }
  })
}

watcher(person, 'type', () => {
  return person.age > 18 ? '御姐' : '萝莉'
})
```

## 参考文献
[深入浅出Vue基于"依赖收集"的响应式原理](https://zhuanlan.zhihu.com/p/29318017)
[剖析Vue原理&实现双向绑定MVVM](https://segmentfault.com/a/1190000006599500)
[Vue源码解析](https://github.com/DDFE/DDFE-blog/issues/7)