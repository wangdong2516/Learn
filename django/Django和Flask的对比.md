## Django和Flask的请求参数的获取

	1. 获取查询字符串数据：
		django: request.GET属性，不区分get请求还是post请求，统一从GET属性获取
				  querydict类型
		
		
		flask: request.args属性，类字典数据，multidict类型
	
		如果需要从查询字符串中获取多个相同的key，使用getlist方法，django和flask一样。
		
	2. 获取表单数据:
		django：request.POST属性
		
		flask: request.form属性 
		 
		如果需要从查询字符串中获取多个相同的key，使用getlist方法，django和flask一样。
		
	3. 获取非表单的请求体数据:
		django: request.body属性，获取到的是原始的二进制数据，需要解码
		
		flask: request.get_data，获取的是原始数据，get_json获取的是json数据
		
		
	4. 获取请求方式:
		
		django: request.method
		
		flask: request.method
	
	5. 获取请求头数据:
		django： request.META，dict类型的数据
		
		flask: request.headers
		
	6. 获取请求路径：
		django: request.path, 不包含域名和参数部分
		
		flask: request.path，不包含域名和参数部分
				request.full_path，包含查询字符串的值
				request.base_url, 获取的是基本的url，包含域名
				
	7. 获取上传的文件
		django: request.FILES属性，是一个类字典对象
		
		flask: request.files属性，字典类型数据
		
	8. 获取请求的用户对象
		django: request.user，获取当前请求的用户对象，未登录为匿名用户
		
		flask: 被flask_login扩展封装为current_user对象
		
	
## Django和Flask响应的构造


				