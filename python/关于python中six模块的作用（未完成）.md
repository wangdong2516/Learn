# 前言
	* six模块的作用就是解决python2和python3中的一些兼容性问题的第三方库，比如解决了urllib的部分方法不兼容，str和bytes类型不兼容的问题
### six.class_types

1. 这里针对的是python2中存在的老式类和新式类，但是python3中之存在新式类的情况
	* 这里简单的介绍一下新式类和经典类的区别：
		1. 写法上的区别：
		
		```
		class A:  			 
			pass
		class B(object):
			pass  
		```
		
		2. 在多继承的时候，新式类采用广度优先搜索，而老式类采用的是深度优先搜索
		3. 新式类更加符合OOP变成思想，统一了python中的类型机制
		4. 在python2中，默认的类都是老式类，除非你显式的继承object，在python3中，默认继承的是object
		5. 新式类可以使用__class__输出本身的类型type(),而老式类使用__class__()输出的和type()输出的结果是不一样的
### six.integer_types
1. 这里针对的是python2和python3中int类型进行了区分，在python2中存在int和long两种整数类型，但是在python3中long被废弃，只包含int类型
### six.string_types
1. 这里针对的是python2和python3中string类型的区分，在python2中使用的是basestring，在python3中，使用的是string
### six.text_type
1. 这里针对的是python2和python3中的文本字符的区别，python2中国使用ASCII码作为默认的编码方式，导致了在python2中string有两种类型str和unicode，在python2中使用的文本字符类型为unicode，在python3中使用的文本类型是str
### six.binary_type
1.	这里针对的是python2和python3中字节序列，在python2中使用的字节序列是str，在python3中使用的字节序列是bytes
### six.MAXSIZE
1. 解决了list，string、dict和其他容器所能支持的最大数量
### six.get_unbound_function(method)
1. 针对python2和python3中unbound function的支持不同，在python2中存在unbound function，在python3中不存在unbound function。
### six.get_method_function(method)
1. 此方法可以在方法对象之外的到函数，在python2中使用的是im_func,在python3中使用的是func
	
			 