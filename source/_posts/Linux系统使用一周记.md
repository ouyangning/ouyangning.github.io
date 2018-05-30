---
title: Linux系统使用一周记
date: 2018-05-30 08:30:50
tags: 周记
---

之前接触`Linux` 系统都是在虚拟机里，第一次用的是`Ubuntu` ，之后又试了试`Debian` 。但是貌似只是在里面试了试几个命令就不了了之了。一来在虚拟机里响应速度比较慢，二来想要打开这个系统得先启动`Windows`，再打开`VMware`， 太笨重了。再者说，当时也实在没有什么切实的需求，所以用起来也不得劲。
不过后来慢慢发现，`Linux` 系统最大的优势在于他对于开发者的友好（买不起苹果），实际体验过后也确实是这样。但同时，也对普通用户十分不友好，如果不能看报错信息，自己解决问题，还是远离`Linux` 系统吧。
## 安装
### Deepin
这一次选择的`Linux`发行版是`Deepin`，想到毕竟是国人开发，应该会比较在意国人的需求。果然，在深度商店里就可以直接安装`Tim`，`WeChat`，`搜狗输入法`等软件。再一个就是，`Deepin`打的口号就是“免除新手痛苦，节省老手时间”，提供了`out-of-box`的使用体验，对我这样的新手可以说是很友好了。

*深度桌面*

![深度桌面](http://op7aviu2v.bkt.clouddn.com/%E6%B7%B1%E5%BA%A6%E6%A1%8C%E9%9D%A2_Desktop_20180530092252.png)

*深度终端*

![深度终端](http://op7aviu2v.bkt.clouddn.com/%E6%B7%B1%E5%BA%A6%E7%BB%88%E7%AB%AF_deepin-terminal_20180530092023.png)

*深度启动器*

![深度启动器](http://op7aviu2v.bkt.clouddn.com/%E6%B7%B1%E5%BA%A6%E5%90%AF%E5%8A%A8%E5%99%A8_dde-launcher_20180530092326.png)

*深度文件管理*

![深度文件管理](http://op7aviu2v.bkt.clouddn.com/%E6%B7%B1%E5%BA%A6%E6%96%87%E4%BB%B6%E7%AE%A1%E7%90%86_dde-file-manager_20180530092440.png)

### 使用U盘安装
因为非常舍不得`Windows`，所以选择`Windows`和`Deepin`双系统，打游戏在`Windows`上，开发在`Deepin`上。先在`Windows`上用`磁盘管理`释放了50G可用空间，打算20G挂载在`/`目录下，30G挂载在`~`目录下。之后在`Windows`上

1. 下载深度启动盘制作工具和官方最新的镜像文件
2. 插入U盘到电脑的USB接口中
3. 打开深度启动盘制作工具
4. 选择深度操作系统镜像文件以及分区
5. 点击`开始`即可制作
6. 进入电脑`BOIS`界面将U盘设置为第一启动项，关闭安全启动，重启电脑
7. 之后就进入`Deepin`的安装界面了

### 工具推荐
所推荐的这些工具，都是基于个人偏好和工作所需，并不一定适合所有人，各取所需就好。
#### miniconda
`conda`是类似于`npm`或`maven`的包管理工具，只是`conda`是针对`python`的。可以安装`miniconda`或`anaconda`进行安装，`anaconda`是一个用于科学计算的`python`发行版，包含了众多流行的科学计算、数据分析的`python`包。`miniconda`是一个`anaconda`的轻量级替代，默认只包含了`python`和`conda`，但是可以通过`pip`和`conda`来安装所需要的包。
##### 安装
可以通过[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/)下载，比从官网直接下载会快一些。下载之后双击打开就可以安装了。
##### 使用
`conda list`列出当前环境下的所有包
`conda list -n env_name`列出一个虚拟环境下的所有包
`conda install package`为当前环境安装某个包
`conda install -n env_name package_name`为某个虚拟环境安装某个包
`conda env list`列出所有`conda`下的虚拟环境
`conda create --name env_name python=x.x`创建指定`python`版本的虚拟环境
`source activate env_name`进入某个虚拟环境
`conda env export > environment.yml`生成当前的环境配置
`conda remove --name env_name --all`删除某个环境
`source deactivate`退出某个环境
#### vim-nox
之所以会提到这个，是因为`Deepin`原生的`vim`不支持`python2`和`python3`，需要重新编译。但是编译起来太麻烦了，刚好`vim-nox`支持多种脚本语言，所以就直接用这个好了。
#### autojump
`autojump`插件可以用于直达常用目录。也可以手动设置权重数据库，很方便。
##### 安装
1. `git clone git://github.com/joelthelion/autojump.git`
2. `cd autojump
./install.py or ./uninstall.py`
##### 使用
`j dir`直达转到对应的目录去
`autojump -a [dir]`在数据库中添加一个目录
`autojump -i [value]`提升当前目录的权重
`autojump -d [value]`降低当前目录的权重
`autojump -s `显示数据库中的统计数据
`autojump --purge` 清除不再需要的目录
