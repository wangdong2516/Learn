
#### 
	在面向对象(OOP)编程中，很多情况下会遇到多继承和多重继承的问题和坑，这里带对大家
	认识一下其中的一个钻石继承(菱形继承)的问题。

什么时候会出现钻石继承(菱形继承)呢？
	
	当在类树中，由多个类共享同一个父类的时候，钻石继承就出现了。
	
为什么会出现钻石继承(菱形继承)呢？
	
	这里先带大家理解一下python中关于继承的一些概念
	
	1. 多继承：多继承就是在声明类的时候，在类名后面的括号中出现一个以上的父类，这
		中情况就叫做多继承。
		也就是一个子类继承一个以上的父类
	
	2. 在python中，因为多继承的存在，就会导致出现钻石继承的问题，这里举个例子说明

``python

	class Super(object):

	    def __init__(self, name):
	        print('Super类的初始化方法开始执行')
	        self.name = name
	        print('Super类的初始化方法执行完毕')


	class Parent1(Super):

	    def __init__(self, name):
	        print('Parent1类的初始化方法开始执行')
	        Super.__init__(self, name)
	        print('Parent1类的初始化方法执行完毕')


	class Parent2(Super):

	    def __init__(self, name):
	        print('Parent2类的初始化方法开始执行')
	        Super.__init__(self, name)
	        print('Parent2类的初始化方法执行完毕')


	class Son(Parent1, Parent2):
	
	    def __init__(self, name):
	        print('Son类的初始化方法开始执行')
	        Parent1.__init__(self, name)
	        Parent2.__init__(self, name)
	        print('Son类的初始化方法执行完毕')
	
	
	son = Son('spam')
	
	
打印结果如下：

	Son类的初始化方法开始执行
	Parent1类的初始化方法开始执行
	Super类的初始化方法开始执行
	Super类的初始化方法执行完毕
	Parent1类的初始化方法执行完毕
	Parent2类的初始化方法开始执行
	Super类的初始化方法开始执行
	Super类的初始化方法执行完毕
	Parent2类的初始化方法执行完毕
	Son类的初始化方法执行完毕


在这里我们可以很清楚的看到，Super类的构造函数被调用了两次，这就会出现问题了，如果
在某些应用场景下，Super的构造函数是一个计数器，那么就会导致错误的结果了。


那么怎么解决钻石继承的问题呢？
	
	为了解决这个问题，python中专门引入了MRO顺序来解决这个问题。
	MRO顺序本质上执行的是广度优先搜索，从左到右，搜索完同一层级的时候，向上爬升。
	保证了每个类中的方法只会被执行一次。避免了同一个类被调用多次的情况。


查看MRO顺序
	
	类名.__mro__
	
	(<class '__main__.Son'>, <class '__main__.Parent1'>, <class '__main__.Parent2'>, <class '__main__.Super'>, <class 'object'>)

这里就引出了python中super()函数的作用了
	
	1. super()函数是用来调用父类的一个方法，是为了解决多重继承的问题的
	2. 使用super()函数调用的是在mro顺序中的直接父类


			
	
