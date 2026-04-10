# breed+pavadan路由器



## 参考链接：

[红米ac2100路由器刷breed和openwrt教程 - 海浪博客 - 博客园](https://www.cnblogs.com/yecss/p/18432303)

[红米ac2100刷breed和openwrt-CSDN博客](https://blog.csdn.net/qq_29781403/article/details/151009974)



## 准备工作

红米AC2100一台

网线两根

openwrt固件以及breed下载：https://yecss.lanzoul.com/i1N0H2aup7te



## 正文：

#### 固件先降级：

确保官方系统降级到2.0.7的版本

降级包所在目录：

![image-20260323224915939](breed+pavadan%E8%B7%AF%E7%94%B1%E5%99%A8.assets/image-20260323224915939.png)

浏览器输 192.168.31.1 进入后台→常用设置→系统状态→手动升级→加载固件(可以保留数据)→开始升级

#### 刷写breed：

降级成功之后再次进入后台，在地址栏获取stok。

复制修改好stok的代码，粘贴到浏览器地址栏，然后**回车**。

若浏览器显示”{"code":0}“，则说明成功。

#### 检查坏块

```shell
http://192.168.31.1/cgi-bin/luci/;stok=把复制的stok粘贴到这里/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=%0A%5B%20-z%20%22%24(dmesg%20%7C%20grep%20ESMT)%22%20%5D%20%26%26%20B%3D%22Toshiba%22%20%7C%7C%20B%3D%22ESMT%22%0Auci%20set%20wireless.%24(uci%20show%20wireless%20%7C%20awk%20-F%20'.'%20'%2Fwl1%2F%20%7Bprint%20%242%7D').ssid%3D%22%24B%20%24(dmesg%20%7C%20awk%20'%2FBad%2F%20%7Bprint%20%245%7D')%22%0A%2Fetc%2Finit.d%2Fnetwork%20restart%0A
```

我坏了三个块，分别是14 15 718，但是依旧可以烧breed

#### 刷breed

```b
http://192.168.31.1/cgi-bin/luci/;stok=把复制的stok粘贴到这里/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=%0Acd%20%2Ftmp%0Acurl%20-o%20B%20-O%20https%3A%2F%2Fbreed.hackpascal.net%2Fr1286%2520%255b2020-10-09%255d%2Fbreed-mt7621-xiaomi-r3g.bin%20-k%20-g%0A%5B%20-z%20%22%24(sha256sum%20B%20%7C%20grep%20242d42eb5f5aaa67ddc9c1baf1acdf58d289e3f792adfdd77b589b9dc71eff85)%22%20%5D%20%7C%7C%20mtd%20-r%20write%20B%20Bootloader%0A
```

这里试了一下，烧录不进去，用url解码器看了一下，是breed官网那边没有了。

![image-20260323225220228](breed+pavadan%E8%B7%AF%E7%94%B1%E5%99%A8.assets/image-20260323225220228.png)



可以用自己的电脑开一个直链，用于下载这个bin文件，如果文件是一样的话后面的加密不需要改，否则需要更改后面的值。

这里我直接在腾讯云服务器上自己部署了一个直链

可以用python直接跑

```python
python -m http.server 9003
```

然后将地址填到上面后编码，可以只替换curl后面的网页即可。

参考：

```shell
http://192.168.31.1/cgi-bin/luci/;stok=16742c3b021146a8bcd4e58ac528b805/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=%0Acd%20%2Ftmp%0Acurl%20-o%20B%20-O%20http%3A%2F%2Fip%3A8080%2Fbreed-mt7621-xiaomi-r3g.bin%20-k%20-g%0A%5B%20-z%20%22%24(sha256sum%20B%20%7C%20grep%20242d42eb5f5aaa67ddc9c1baf1acdf58d289e3f792adfdd77b589b9dc71eff85)%22%20%5D%20%7C%7C%20mtd%20-r%20write%20B%20Bootloader%0A
```

后续在论坛上还找到别的

```shell
http://192.168.31.1/cgi-bin/luci/;stok=CCCCCCCCCCC/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=%0Acd%20%2Ftmp%0Acurl%20-o%20B%20https%3A%2F%2Fbreed.hackpascal.net%2Fr1416%2520%255b2022-07-24%255d%2Fbreed-mt7621-xiaomi-r3g.bin%20-k%20-g%0Amtd%20-r%20write%20B%20Bootloader%0A
```







#### 查看刷写breed结果

浏览器输入刷写回车后只能靠灯的状态观察结果，路由器在60秒内重启则代表刷BREED成功，灯会从蓝 -> 橘 -> 蓝 最终进入系统，breed下载服务正常且网速够快，下载过程几乎无感，

如果灯一直是蓝，没有变橘那就是breed没有下载成功 或者 没有校验成功

等待完全重启进入系统，再进行下一步操作。

接着使用网线一端连接路由器Lan口，另一端连接电脑

之后拔掉电源，然后按住reset（电源口旁边的小孔）的同时插上电源，按住四五秒之后如果system指示灯是蓝色闪烁说明进入breed，然后打开浏览器的无痕模式（不同浏览器说法可能不一样），输入192.168.1.1即可进入breed后台。



#### 更新Breed

后续刷入固件，可能会因为breed版本老旧导致没有合适的刷入选项，需要更新成下面这个breed版本

![image-20260323230011197](breed+pavadan%E8%B7%AF%E7%94%B1%E5%99%A8.assets/image-20260323230011197.png)





![image-20260323230014966](breed+pavadan%E8%B7%AF%E7%94%B1%E5%99%A8.assets/image-20260323230014966.png)

手动重启，重新进入breed



这里吧breed整好之后就可以随意烧别的系统了，比如openort或者padavan，我因为要用华工校园网，这里选择自行编译padavan的方案

参考教程

[小米路由器3 SCUT校园网刷scut-padavan固件方法_scutclient-CSDN博客](https://blog.csdn.net/benobug/article/details/120582238)

padavan有提供studentclient这个插件，所以就可以连接上了

这里有AC2100编译好的trx文件，同款型号直接用即可

https://pan.quark.cn/s/3257c5c5eb7f?pwd=pY4X



按照教程步骤配置好，就可以使用了（懒得写了...）

