# RxSwift和RxCocoa入门
> 本文翻译于raywenderlich，作者Ellen Shapiro，原文地址是https://www.raywenderlich.com/138547/getting-started-with-rxswift-and-rxcocoa

---

代码能按你的意愿执行总是件很棒的事。在程序中改变一些东西，告诉代码，然后它就照做了。嗯，这是坨好代码！

在面向对象时代，大多数程序都像这样运行：你的代码告诉你的程序需要做什么，并且有很多方法来监听变化–但同时你又必须主动告诉系统什么时候发生的变化。

目前为止还是很不错，不过如果变化发生时，代码能自动响应更新，生活是不是会更美好呢？这就是响应式编程的基本思想：你的程序可以对底层数据的变化做出响应，而不需要你直接告诉它。这样，你可以更专注于所需要处理的业务逻辑，而不需要去维护特定的状态。

在Objective-C和Swift都可以实现这种操作，主要是通过Key-value Observation，在Swift中还可以使用setter或didSet方法。不过，有时这些方法并不那么好使。为了避免这些问题，现在市面上已经有一些Objective-C和Swift的框架，来实现响应式编程。