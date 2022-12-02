# navicat 的使用笔记

## 创建查询

---

1. 创建表格

   ```sql
   CREATE TABLE student(
   stuid int not null auto_increment PRIMARY KEY,
   stuname varchar(100),
   stustatus varchar(100)
   );
   ```

2. 插入数据

   ```sql
       insert into student (stuname,stustatus) value ("koliko", "nomal");
   ```

   > 插入多组数据只要多几个括号就行

3. 删除数据

   ```sql
   delete from student WHERE stuid = 1;
   ```

4. 修改数据

   ```sql
   UPDATE student set stuname = "kolikoko" , stustatus = "nice" where stuid = 2 ;
   ```

5. 查找数据

   ```sql
   SELECT * FROM student;
   SELECT * FROM student where stuid = 2;
   SELECT stuname, stustatus from student where stuid = 2;
   ```

6. 查询信息条数

   ```sql
   SELECT COUNT(*) from student;
   ```

7.  删除表

    ```sql
    truncate table student;//删除表，重置自增
    delete from student;//删除表，不重置
    drop table student;//销毁表
    ```

8. 覆盖信息(适合表格，覆盖主键的信息)

   ```sql
   REPLACE student ( stu_id, stu_name, stu_phone ) VALUES ( "222","che","123");
   ```


9. 给查找的数据表新建一个序号排序

   ```sql
   SELECT (@i:=@i+1) stu_count, s.*  FROM student s, (SELECT @i:=0) AS stu_count10.给查找的数据一个排序要求
   ```

10. 给查找的数据表一个排序要求

    ```sql 
    SELECT * FROM student WHERE stu_first="新媒体部" or stu_second="新媒体部" ORDER BY stu_first;
    ```

    
