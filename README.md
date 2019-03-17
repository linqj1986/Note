# Note

1.AOP-面向切面编程

设计目的：解决只有单继承的语言缺点。

优点：不入侵原有代码。

实用项目：MLeaksFinder，实现方式是load中更换UIViewController等的viewDidDisappear，在新方法中进行检查。

只要pod进工程即可，不需要修改原有代码。

