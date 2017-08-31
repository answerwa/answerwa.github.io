title: python3和python2的差别
date: 2015-01-13 23:21:46
tags: python
---
学习python已经有段时间了，断断续续的，但是基本上每天也会多多少少看看。  
由于学的是python3，因此经常都会遇到一些兼容性的问题。其实也不可能说一直盯着官方文档看完，所以还是遇到问题后再解决，但是好多都是当时遇到查一下，解决了就忘了，今天新开一篇博客，以后遇到了，可以记录一下。  
推荐一下[廖学峰老师的网站](http://www.liaoxuefeng.com/ "廖学峰")，git和python都是在这个网站学习的。本篇博客可能会引用部分代码。
## print  
从python3开始，print变成了一个函数，因此最直观的说明就是，在使用它时，需要加括号了。  
```python
str = 'there are test code'
print(str)
```
格式化输出也是同样的。
```python
i = 3
print('this is the %dth blog' % i)
```
ps：其实，简单说来，就是在python2的基础上，用括号把它包起来（个人理解）。
## bytes str
这是在学习socket时遇到的一个问题。主要是由于在python3中对文本和二进制数据作了明确的区分，文本总是Unicode，由str类型表示，二进制数据则由bytes类型表示。
在python2中，connect后，send时，是发送的字符串，代码如下：
```python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('www.sina.com.cn', 80))
s.send('GET / HTTP/1.1\r\nHost: www.sina.com.cn\r\nConnection: close\r\n\r\n')
```
而在python3中，send时，需要发送bytes([socket.send(bytes[, flags])](https://docs.python.org/3/library/socket.html "socket")),代码修改起来也比较简单，将'abc'改为b'abc'就可以了，代码如下：
```python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('www.sina.com.cn', 80))
s.send(b'GET / HTTP/1.1\r\nHost: www.sina.com.cn\r\nConnection: close\r\n\r\n')
```
当然，正规的方法肯定不是这样的，str与bytes的转换关系如下：
> str-----encode()--->bytes  
> bytes---decode()--->str  

```python
>>> by1 = b'test'
>>> type(by1)
<class 'bytes'>
>>> str1 = by1.decode('utf-8')
>>> str1
'test'
>>> type(str1)
<class 'str'>
>>> by2 = str1.encode('utf-8')
>>> by2
b'test'
>>> type(by2)
<class 'bytes'>
```
另外拓展一个[链接](http://www.ituring.com.cn/article/61192 "")吧。
## 图形编程 tkinter
这个目前用的不多，单纯记录一下：
> Tkinter → tkinter  
> tkMessageBox → tkinter.messagebox  
> tkColorChooser → tkinter.colorchooser  
> tkFileDialog → tkinter.filedialog  
> tkCommonDialog → tkinter.commondialog  
> tkSimpleDialog → tkinter.simpledialog  
> tkFont → tkinter.font  
> Tkdnd → tkinter.dnd  
> ScrolledText → tkinter.scrolledtext  
> Tix → tkinter.tix  
> ttk → tkinter.ttk  