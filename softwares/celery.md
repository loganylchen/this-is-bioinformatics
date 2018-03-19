# [Celery](http://docs.celeryproject.org/en/latest/index.html)



## 安装

### windows 安装


#### windows中的坑

1. **celery**在发展的过程，越发觉得windows上缺少的东西太多，所以随着版本更新，就不在支持windows，目前在windows上安装`celery==3.1.25`。
2. 如果要连接数据库进行`backend`设置，由于`python3`不在支持`MySQLdb`，所以需要用`PyMySQL`替代，在使用时需要做如下导入。

```python
import pymysql
pymysql.install_as_MySQLdb()
```
3. 连接数据库进行更新的时候，还会产生`Incorrect string value`的错误。是返回的result的字符串填写到mysql中的时候就变成奇怪的字符串了。