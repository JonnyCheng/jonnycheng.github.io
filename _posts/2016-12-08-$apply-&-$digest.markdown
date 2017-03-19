---
layout: post
title:  "$apply & $digest"
date:   2016-12-08 +0800
categories: angular
---

$apply()与$digest()是AngularJS的两个核心功能。了解AngularJS的工作原理首先需要了解$apply()与$digest()是如何工作的。

#### $apply & $digest

AngularJS提供了众所周知的双向绑定特性，大大简化了我们的生活。数据绑定意味着改变视图的同时，scope会自动更新。简而言之，scope与视图同步改变。AngularJS是如何做到的？当你写了一个表达式{{aModel}}后AngularJS会在scope上帮你设置一个watcher用来更新视图。这个watcher就像你在AngularJS中设置的一样：
{% highlight javascript %}
$scope.$watch('aModel', function(newValue, oldValue) {
  //使用新值更新视图
});
{% endhighlight %}
$watch的第二个参数就是监听函数，并且在aModel值改变时被调用。这里很好理解回调的触发，但还有大问题就是AngularJS是如何判断何时被触发的。换句话说，AngularJS是如何知道当aModel改变时它能触发对应的监听。难道它会定期去检查一下scope值是否改变吗？好吧，该$digest循环登场了。

watcher结束后$digest就开始工作了。watcher结束后，AnglarJS检查scope模型，如果有变化，相应的回调函数则被调用。所以，我们下一个问题就是$digest循环在何时以何种方式开始。

$digest循环是以$scope.$digest()的方式调用。假定你通过ng-click指令改变了scope。这种情况下，AngularJS会自动通过调用$digest()方法激发$digest循环。当$digest循环开始，它将关掉所有watcher。当当前scope值与之前计算的值不同时这些watcher才会检查。如果是这种情况，相应的监听函数开始工作，同是视图也会做相应的更新。除ng-click外，ng-model,$timeout等都会自动触发$digest循环。

到目前为止我们了解了一点，以上的情况，AngularJS没有直接调用$digest()方法。取而代之的是，调用了$scope.$apply()，进而又调用了$rootScope.$digest()。结果就是，digest循环始于$rootScope，随后遍历了所有子节点。

现在让我们假定你给你一个button附加一个ng-click指令。当button被点击时，AngularJS用$scope.$apply()包装这个函数调用。所以，当函数正常执行，改变模型，$digest循环便启动以保你的改变可以展现在视图层上。

注：$scope.$apply()自动调用$rootScope.$digest()。$apply()有两种类型。一种是将函数作为参数，监听后出发$digest循环，另一种是不带任何参数，当被调用时就开始$digest循环。接下来我们一起研究一下为什么说前者是更好的方式。

#### 何时手动调用$apply()

如果AngularJS经常在$apply()中包装我们的代码，并且启动$digest循环，那么何时需要我们手动调用$apply()呢？实际上，AngularJS设计的很清楚。它只负责存在于AngularJS上下文中的模型改变，也就是包装在$apply()中的代码。Angular内建指令已经达成这个功能。然而，如果你改变了Angular上下文以外的模型，那你需要通过手动调用$apply()来通知Angular变化。就类似于你要告诉Angular你正在改变模型，它应该停掉watcher。

例如，如果你使用JavaScript的setTimeout()方法来更新一个scope模型，Angular不知道你会改变什么。这时就需要你去手动调用$apply()，来出发$digest循环。类似，如果你写了一个指令来设置DOM时间监听，并在handler中改变一些模型，你需要调用$apply()确保这些改变生效。

我们来看个例子。假如你有个页面，当页面加载完毕后你想在2秒后展示一条信息。你的实现可能是一下的样子。
{% highlight html %}
<body ng-app="myApp">
   <div ng-controller="MessageController">
     Delayed Message: {{message}}
   </div>  
</body>
{% endhighlight %}
{% highlight javascript %}
angular.module('myApp',[]).controller('MessageController', function($scope) {
    
      $scope.getMessage = function() {
        setTimeout(function() {
          $scope.message = 'Fetched after 3 seconds';
          console.log('message:'+$scope.message);
        }, 2000);
      }
      
      $scope.getMessage();
    
    });
{% endhighlight %}
运行程序，你会发现两秒后延时方法运行，并且更新了message。然而，视图却没有更新。原因你可能已经猜到，就是我们忘记去手动调用$apply()。因此，我们需要一下的方式去更新我们的getMessage()方法。
{% highlight javascript %}
angular.module('myApp',[]).controller('MessageController', function($scope) {
    
      $scope.getMessage = function() {
        setTimeout(function() {
          $scope.$apply(function() {
            $scope.message = 'Fetched after 3 seconds'; 
            console.log('message:' + $scope.message);
          });
        }, 2000);
      }
      
      $scope.getMessage();
    
    });
{% endhighlight %}
如果你运行了这个例子，你可以看到视图将在两秒后更新。唯一的改变就是我们用$scope.$apply()包装的代码自动触发了$rootScope.$digest()。

注：此处应该使用$timeout，不需要再需要手动$apply()。

#### $digest loop循环多少次

当$digest循环运行，如果模型有改动，watcher则被执行。这就引出一个很重要的问题。假如一个监听方法自己改变了scope模型该怎么办？AngularJS会怎么处理？

答案就是$digest循环不会只运行一次。在当前循环结束结尾，它会重新启动去检查是否还有模型有改动。这就是基本的脏检查，所以$digest循环会一直循环直到没有模型变化为止，直到达到最大循环次数10次。

#### 总结

希望这篇文章可以说明$apply与$digest。重要的是了解什么情况下Angular才能检测到改变，如果不能你可以手动$apply()。
