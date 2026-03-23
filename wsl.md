# WSL使用



## 代理问题

终端出现：

wsl: 检测到 localhost 代理配置，但未镜像到 WSL。NAT 模式下的 WSL 不支持 localhost 代理。

### 解决方案

**开启镜像网络模式**

镜像模式让 WSL 直接共享 Windows 主机的网络接口和 IP 地址，这能完美解决代理无法同步、端口转发困难等一系列“陈年旧疾”。

1. 在 Windows 中按下 `Win + R`，输入 `%USERPROFILE%` 并回车。
2. 在打开的文件夹中寻找名为 `.wslconfig` 的文件。如果没有，请手动创建一个（注意后缀名必须是 `.wslconfig`，不要带 `.txt`）。
3. 将以下配置复制进去并保存：

```shel
[wsl2]
# 开启镜像网络模式
networkingMode=mirrored

[experimental]
# 自动同步 Windows 的代理设置到 WSL (此项建议放在实验性分组下)
autoProxy=true
```

4. 保存文件后，在 PowerShell 中执行以下命令重启 WSL：

```
wsl --shutdown
```

5. 重新打开 WSL 终端。