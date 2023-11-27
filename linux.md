# linux学习



## 命令



### ls

列出目录内的内容

```
ls [-a -l -h] [linux路径]
```

-a 显示隐藏文件

-l  纵向展示

-h 通过易读的方式展现，比如文件大小带单位

可以组合使用



## cd/pwd

cd:change directory 切换路径

pwd:print work directory 显示当前路径



## mkdir

make directory 创建文件夹

```--
mkdir [-p] linux路径
```

-p 选项可选，表示自动创建不存在的父目录，适用于连续多层



## touch-cat-more

touch 创建文件

cat 查看文件内容

more 可翻页查看内容，空格或者回车翻页，使用b回翻，按q推出



## ln软连接

```shell
ln -s 参数1（被链接的文件或文件夹） 参数2（要链接qu'de）
```

