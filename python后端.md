# python后端



## 连接数据库

这里我使用的是PyMySQL

可以参照这里来配置环境[Python连接MySQL数据库方法介绍](https://zhuanlan.zhihu.com/p/79021906)



### 使用步骤

1. 导包`import pymysql`

2. 连接数据库

   ```python
   # 连接数据库
   conn = pymysql.connect(host='127.0.0.1',  # 连接名称，默认127.0.0.1
                          user='root',  # 用户名
                          passwd='abc12456',  # 密码
                          port=3306,  # 端口，默认为3306
                          db='user',  # 数据库名称
                          charset='utf8'  # 字符编码
                          )
   ```

3. 基本使用

   ```python
   cur = conn.cursor()  # 生成游标对象
   sql = "select * from user "  # SQL语句
   cur.execute(sql)  # 执行SQL语句
   data = cur.fetchall()  # 通过fetchall方法获得数据
   for i in data[:2]:  # 打印输出前2条数据
       print(i)
   cur.close()  # 关闭游标
   conn.close()  # 关闭连接
   ```

**注意：**他会有时候报错`pip install cryptography`，可能是环境问题,在终端输入`pip install cryptography`



### PyMySQL防注入

在使用的时候别忘了防注入

这里参考[python与pymysql交互 防止sql注入](https://blog.csdn.net/erfan_lang/article/details/120527431)

主要的操作是使用占位符`%S`

比如下面的操作

```python
#游标
cursor = conn.cursor(pymysql.cursors.DictCursor)
#默认游标取出来的数据是元组((),)
#DictCursor对应的数据结构{[]，]，如果用的是fetcheone，那么结果是{}
sql = "select * from students(name,age,gender,c_id) values(%s,%s,%s,%s);"

try:
    #执行sql语句的传入的参数，参数类型可以是元组、列表、字典
    ret = cursor.execute(sql,["黄蓉",75，"男",3])
    print(ret)
    # 增删改都必须进行提交操作(commit)
    conn.commit(sql)
except  Exception as e:
    #对插入、修改、删除的数据进行撤销，表示数据回滚（回到没有修改数据之前的状态）
    conn.rellback
finally:
    #关闭游标
    cursor.close()
    #关闭连接
    conn.close()
```

