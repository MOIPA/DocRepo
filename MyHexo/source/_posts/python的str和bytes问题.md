title: python的str和bytes问题
author: tr
tags:
  - Python
categories:
  - Python
date: 2019-07-18 15:42:00
---
### python字符编码问题

最近在复习python，又遇到了编码问题，下定决心弄懂，把自己的理解写出来。

#### 什么是编码

1. 编码是由于计算机无法识别除了0/1以外的东西而产生的。早起只有英文字母的时代，一个字节即可以表达所有字母，这个编码叫ascall，每个字母通过这个表可以将字母转为对应的二进制数，存在磁盘中。
	
    例如字母B的ascall是66，转为二进制：`0100002`存在磁盘上。如果这时候一个不兼容ascall码表的单字节码表读取的时候认不得这个二进制，就会产生乱码。
    
2. 后来由于发展，ascall码不够用，出现了很多码表，其中以一种规范为unicode（万国码），这个表里英文是2字节表示，中文3字节，不兼容ascall码，注意的是unicode不是物理存储实现，所以需要一种存储方式和ascall的存储方式兼容。
	
    所以又有人推出了 utf-8规范，这个规范是对unicode的实现，具体做法是对unicode的对应字符序号（码点：code-point）使用边长字节进行存放。内存中unicode是2字节存放码点。
    
    在python中显示验证：
    
    a变量存放了str类型的unicode的中文，在python3里头unicode的字自动会转为中文的，类型都是str。这里将str编码为utf-8，a就是这3字节（24位）的数。再使用gbk编码解码读取，gbk不兼容utf-8显然无法识别这串二进制数。
    
    ```python
    >>> a = '\u8bf7'
    >>> a
    '请'
    >>> a.encode('utf-8')
    b'\xe8\xaf\xb7'
    >>> a= a.encode('utf-8')
    >>> a.decode('gbk')
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    UnicodeDecodeError: 'gbk' codec can't decode byte 0xb7 in position 2: incomplete multibyte sequence
    
    ```
    
#### python的unicode和str区别

1. str在python2.7里指向的是二进制，在python3里是unicode字符（统一了str和unicode,取消了unicode对象），2.7 print时会转为unicode显示，因为内存默认读取unicode编码。

	python3 str为unicode字符集，传输需要编码：
    
	str ==> encode：bytes
    
	bytes ==> decode：str

2. python2 里unicode是文本字符串，str是字节字符串！！！能直接看到str的二进制表示，看不到unicode对象的二进制内存表示！unicode对象和str对象之间转换需要encode和decode，内存里默认的用unicode编码但是存放的是unicode的序号（code-point）的二进制，所以str的二进制解码就是unicode字符
	
    ```python
	>>> a = '唐'
	>>> a
    '\xe5\x94\x90'
    >>> a.decode('utf-8')
    u'\u5510'
    >>> 

    ```

	python2将字节字符串和文本字符串视为一致了，这是一个设计错误，所以在3里取消了！
    
    python会自动处理他们的关系。例如将unicode对象和str对象拼接时，python自动对str进行转换，转为unicode，但是转换的decode方式为默认的ascall，但这年头谁还纯用ascall，所以非常容易出错，很烦。
    
    例如：a为str对象，b为unicode对象，两者拼接，str对象是二进制，需要将str转为unicode，默认执行str.decode('ascall')，但是实际文本是utf-8的二进制，所以出错。

3. （2.7）`str`是python内存的一种二进制编码（但是编码方式取决于环境：如果是在解释器中写，二进制为终端的编码类型，文件的话则是文件保存编码类型），python读取这个str对象（二进制序列）就知道是什么字符，显示到显示器。

	`unicode`对象是一种字符集并非二进制（本质在内存中是2个字节标识的序号二进制），如果要存储`unicode`对象到磁盘，需要自己指定编码方式如`utf-8`（以三个字节表示中文），如果指定的编码方式如`gbk`会无法写入，因为`gbk`不支持`unicode`编码。
    
    但是在python中是无法直接看到`unicode`对象的二进制数。（注意的是在`python3`中`unicode`和`str`都为`str`类型）

	（2.7）举例子： ‘唐’ 如果是str类型 且在终端的解释器环境下为gbk，那么`a = '唐'` `a`在内存里就是`gbk`的二进制数组。再次打印 `a` 时计算机读取内存的二进制，根据终端的码表显示，如果写的时候终端为`utf-8`,打印的时候改成`gbk`那么应该会乱码。
    
    在python中验证乱码：
    
    ```python
    >>> a = '唐'
    >>> a 	#python2.7里头直接打印a打印的是a指向的内存的二进制数，需要print函数
    '\xe5\x94\x90' # 内存里头的二进制数，因为终端是utf-8所以是三个字节24位
    >>> type(a)
    <type 'str'>
    >>> print(a)
    唐
    # 这时候我将终端的编码改为了gbk
    >>> print(a)
    鍞�
    # 乱码，因为gbk读取这个24位读不懂 

    ```
    
    
#### 使用py2需要注意的点（参考了StackOverflow)
 
 1. 所有文本字符串都应该是unicode类型，而不是str类型。如果处理的是文本，而变量类型是str，这就是bug了！(因为在内存中默认的是unicode，磁盘中才是对于编码后的二进制)

 2. 若要将字节串解码成字符串，需要使用正确的解码，即 var.decode(encoding)（如， var.decode('utf-8') ）。将文本字符串编码成字节，使用var.encode(encoding)。

 3. 永远不要对unicode字符串使用 str() （str转为了二进制，编码方式取决于环境），也不要在不指定编码的情况下就对字节串使用 unicode() 。
 
 4. 当应用从外部（磁盘，网络）读取数据时，应将其视为字节串，即str类型的，接着调用.decode() 将其解释成文本。同样，在将文本发送到外部时，总是对文本调用.encode() 。