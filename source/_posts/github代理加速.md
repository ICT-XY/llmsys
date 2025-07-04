---
title: github代理加速
date: 2025-06-16 22:33:32
tags: 基础技能
---

# github 代理加速办法

## 方法1：地址加前缀：
例如原网址:
```
https://github.com/Dao-AILab/flash-attention.git
```
代理为：
```
https://ghfast.top/
```
则在clone仓库到本地时，使用如下写法：
```
git clone https://ghfast.top/https://github.com/Dao-AILab/flash-attention.git
```

## 方法2：设置环境变量
法1每次都要手动加前缀地址，且必须显式指定，比较受限。
比如有的仓库有很多子模块，法1在拉取子模块的时候不会自动加前置，导致子模块的拉取并没有被加速。
上述问题可以通过设置环境变量解决,""中为代理地址：
```
git config --global url."https://ghfast.top/".insteadOf https://
```

## 方法3：写入配置文件
在~/.gitconfig中增加如下配置即可：

```
[url "https://ghfast.top/https://github.com/"]
    insteadOf = https://github.com/
```


## 可用代理列表：
没有一个一个试，目前只用了第一个
```
https://ghfast.top/

https://gh.api.99988866.xyz

https://ghp.ml1.one

https://doget.nocsdn.com

https://ghgo.xyz

https://slink.ltd

https://ghps.cc

https://gh-proxy.com

https://ghproxy.cc

https://cf.ghproxy.cc

https://github.tmby.shop

https://gh-proxy.ygxz.in

https://ghproxy.net

https://ghproxy.cn
```

## 参考：
https://blog.csdn.net/qq_39749966/article/details/145152772
https://blog.csdn.net/supperman_009/article/details/130466946