# windows 下编码错误
```sh
版权归作者所有，任何形式转载请联系作者。
作者：petanne（来自豆瓣）
来源：https://www.douban.com/note/521708832/

问题是使用pip install django 或其它时，始终报错：
UnicodeDecodeError: 'ascii' codec can't decode byte 0xb1 inposition 34: ordinal
not in range(128)

原因是windows的cmd环境默认为gbk编码，pip默认用utf8编码。
在Linux和Mac中，terminal环境默认的是utf8编码，所以不会报错。

解决方法：
python目录 Python27\Lib\site-packages 建一个文件sitecustomize.py 
内容为： 
import sys 
sys.setdefaultencoding('gbk') 
```