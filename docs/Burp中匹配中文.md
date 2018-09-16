## Burp中匹配中文

**问题：**Burp中有很多地方可以进行正则匹配，比如Instruder模块中筛选响应包，proxy模块中匹并配替换字符串。中文在匹配的时候，添加进匹配列表就变身了，关键是与数据包内的相应字符不能匹配。

**解决办法：**

1. 在user option中设置字符集(character sets)为显示原始字节流(Display as raw bytes);

![image1](..\images\image1.png)

2. 在响应包中复制要匹配的中文，显示的是乱码；

![image2](..\images\image2.png)



3. 将复制的乱码粘贴到添加匹配字符串的地方。

![image3](..\images\image3.png)