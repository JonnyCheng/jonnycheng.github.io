---
layout: post
title:  "浅谈React Diff算法"
date:   2016-02-01 +0800
categories: development
---
React是Facebook&Instagram开发构建用户界面的类库。开发React的初衷是为了解决大型复杂UI应用性能问题。接下来，我会浅谈一下React是如何完成这项任务的。

虚拟DOM

React中，render的结果仅仅是轻量级的JavaScript对象，将之称为虚拟DOM。

为何要使用虚拟DOM？只是因为真实DOM元素冗余。举个例子：

![真实DOM](/img/1.jpg)

因此相比操作真实DOM，操作JavaScript对象简单得多也会快得多。

Diff算法

React的设计原则是状态决定UI，不同状态相当于不同的页面，通过算法对DOM状态对比进而完成render。

传统Diff算法复杂度需要O(n^3)，这显然不能满足性能要求，React运用了一种极为简单的方法将复杂度降到了O(n)。

当同一节点前后状态不同的时候，React直接删除起始状态的节点，接着插入新建节点。

{% highlight javascript %} 
renderA: <div />
renderB: <span /> 
=> [removeNode <div/>], [insertNode <span />]
{% endhighlight %}

同理，也适用于React组件，先删除起始状态组件，然后插入新建组件。

{% highlight javascript %} 
renderA: <Header />
renderB: <Content />
=> [removeNode <Header />], [insertNode <Content />]
{% endhighlight %}

由此逻辑可知React是通过逐层diff,简单粗暴。

![真实DOM](/img/react1.png)

结束diff过程，组件或节点会根据状态的改变而重新render，这期间DOM也只会更新一次。



