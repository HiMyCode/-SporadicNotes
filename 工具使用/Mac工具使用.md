# 一、安装软件问题

## 1.1、XXX.app 已损坏，打不开。您应该将它移到废纸篓】

将 系统偏好设置 -> 安全性与隐私 -> 通用 -> 允许从以下位置下载应用 改为：任何来源。终端执行命令：

```shell
sudo spctl --master-disable
```

## 1.2、XXX.app 已损坏，打不开。您应该将它移到废纸篓】

终端执行命令：

```shell
sudo xattr -r -d com.apple.quarantine [应用绝对路径名]
```

一般应用绝对路径名位置：/Applications/应用名

若应用有种有空格，使用反斜杠/转义：/Applications/Nav\ Nam.app