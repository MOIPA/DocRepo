title: python
author: tzq
tags: []
categories: []
date: 2020-05-09 19:24:00
---
### python语法

****

1、python对大小写敏感

 定义变量始终使用小写，true或false这种特殊关键字开头大写

***

2、引号使用
 
 ①三引号可定义分开的字符串
 
 	course = '''
	hi john
	here is your first email

	thank you
	'''

	print(course)
    
    
 ②单引号套双引号
 
 	course = 'python for "beginners"'
	print(course)
    
输出python for "beginners"    

③双引号套单引号

	course = "python's beginners"
	print(course)
输出python's beginners


***

3、索引

①负指数索引

	course = "python for beginners"
	print(course[-1])
输出s


②

		course = "python for beginners"
		print(course[0:3])
    
输出pyt (排除字符和索引3)

③

course = "python for beginners"
    print(course[:])
输出python for beginners（默认索引全部）

④

	name='jennifer'
	print(name[1:-1])
    
输出ennife

***

4、花括号定义占位符 打了f以后，引号里的内容会被直接输出，加上花括号可以保证像first这种定义量可以输出你给的赋值

	msg=f'{first} [{last}] is a coder
	print(msg)
    
输出john [smith] is a coder

format格式化函数

	my_list = ['菜鸟教程', 'www.runoob.com']
	print("网站名：{0[0]}, 地址 {0[1]}".format(my_list))  # "0" 是必须的
    
 输出 网站名：菜鸟教程, 地址 www.runoob.com
 
 
 ***
 
 5、替换字符
 
 	course='python for beginners'
	print(course.replace('beginners','absolute beginners'))

输出python for absolute beginners


****

6、find方法 返回字符或字符序列（find大小写敏感）

	course='python for beginners'
	print(course.find('beginners'))

输出11


****

7、in 产生布尔值的

	course='python for beginners'
	print('python' in course)

输出true

****

8、运算

①除法返回整数

	print(10 // 3)

输出3

②除法

	print（10 / 3）
    
输出3.33333


③除法返回余数

	print（10 % 3）
输出1（返回余数）

④立方

	print(10 **3)
输出1000

⑤增广赋值运算

	x=10
    x-=3
    print(x)
    
***

9、一些内置函数

①round

		x=2.4
		print(round(x))
输出2（四舍五入）

②abs绝对值

	print(abs(-2.9))

输出2.9


***

10、引入模块

	import math
	print(math.ceil(2.9))

输出3（ceil向上取整，floor向下取整）


11、if语句

	is_hot=False
	is_cold=False
	if is_hot:
    	print("it's a hot day")
    	print('drink plenty of water')
	elif is_cold:
    	print("it's a cold day")
    	print("wear warm clothes")
	else:
    	print("it's a lovely day")
	print("enjoy your day")   //按shift加tab缩进

输出it's a lovely day

enjoy your day

12、逻辑运算符：and、or、not

	has_high_income=True
	has_criminal_record=False
	
	if has_high_income and not has_criminal_record:
   	 print("eligible for loan")
