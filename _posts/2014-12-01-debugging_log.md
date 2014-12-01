---
layout: post
title: 记一次调试
category: technology
---

方才用了大约1个半小时调了一个直观上很诡异的bug，最终原因是其实是一个很低级的错误（这种事情貌似经常发生。。。）。

故事大概是这样子的：在最近在做的一个系统中，使用了一个动态构建的自动机来维护系统的某些状态，每个自动机状态由一个Set来唯一标识。有一个StateMachine，里面有一个StateSet，包含了若干已创建的状态，还有边啊转移啊一堆别的乱七八糟的。因此每次迭代会有一个函数getCurrentState来获取当前的状态，或者创建一个新的状态，差不多是这样：

<!--break-->

```java
State getCurrentState(Set set) {
    if (set can identify an existing state) {
		return state;
    } else {
		return new State(set);
    }
}
```

然后问题来了，执行的时候发现开始若干次迭代都没有问题，直到有一天，set是一个之前已经出现过的set，应该获得一个已经创建过的状态，可是，它神奇地创建了一个新的状态。

然后就是一阵断点一阵log，发现原因是某一次执行的时候已经创建的某个状态的set被修改了。。。搜了遍又审了遍代码，确定了这个set是绝逼没有被调用过什么setter或者先getter再修改之类的。

于是只能在代码里插了若干log，最后确定修改发生在这样子两行代码里面：

```java
for (IAction action : startNode.getActionsIncludingMenuBack()) {
	weightSumMap.put(action.getDescription(), 0);
	unexecutedWeightMap.put(action.getDescription(), 0);
}
```

这里的startNode是某个状态的名称，别的语义不用管。那么问题在哪呢？函数体里的两行并无关系，所以只能是这个getActionsIncludingMenuBack()的问题了，然后看到实现后，一目了然了。这个函数里大概干了这么件事儿:

```java
...
Set<IAction> actionSet = XXXXSet;
actionSet.add(action1);
actionSet.add(action2);
...
```

并没有对actionSet进行初始化，而是直接把XXXXSet赋值给了它，低级错误。。。而这个XXXXSet就是已经创建的某个状态的set，所以尽管这个函数的意图只是获取，却无意进行了修改，然后bug就来了。

改成

```java
Set<IAction> actionSet = new HashSet<IAction>(XXXXSet)
```

一切就好了。