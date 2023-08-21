# anacodna环境配置

## 一些格外的环境配置帮助

这章用于补充一些环境配置过程中需要用到的操作和工具

## 选择激活的虚拟环境

使用以下命令用于查看conda创建的环境

``` CMD
conda info --envs
```

使用以下命令用于切换进入对应的环境（xxxx请替换为需要激活的环境名称，若删去xxxx则进入base环境）

``` CMD
conda activate xxxx
```

## 配置镜像源

在pip和conda中分别配置镜像源，以提升下载体验

## 配置pip镜像源

首先请升级pip为最新版本

``` CMD
python -m pip install --upgrade pip
```

然后进行镜像源的配置

``` CMD
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

https内容即为镜像源地址
以下是一些常见的镜像源

``` CMD
清华：https://pypi.tuna.tsinghua.edu.cn/simple
阿里云：http://mirrors.aliyun.com/pypi/simple/
中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
华中理工大学：http://pypi.hustunique.com/
山东理工大学：http://pypi.sdutlinux.org/
豆瓣：http://pypi.douban.com/simple/
```

## 配置conda镜像源

``` CMD
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
```

## conda中的pip

在conda中下载pip，以替换所有的pip install 为 conda install，

``` CMD
conda install pip
```

在此之后，任何环境配置在对应的虚拟环境下可以放心大胆的使用 pip install 而不是 conda install了！
