## 1.8.1 安装虚拟环境
在使用 Scrapy、Django、 以及一些大型项目之前，我们通常会安装一个虚拟环境 （也称为 virtualenv），它可以让你计算机上的编码环境保持整洁。 虚拟环境会以项目为单位将我们的项目的 Python 运行环境隔离开。 这意味着对一个网站的修改不会影响到你同时在开发的其他任何一个网站。

### 1. 通过virtualenv创建虚拟环境
首先，我们可以通过 pip 进行安装，命令如下：

`pip install virtualenv`

然后，进行虚拟环境的创建：

```
$ mkdir myproject
$ cd myproject
$ virtualenv venv
```

在这里，我们创建了一个名为myproject的文件夹，然后在这里里面创建了虚拟环境 venv。

但在创建虚拟环境时增加 --no-site-packages 选项的virtualenv便不会读取系统包，比如：

`virtualenv nowamagic_venv --no-site-packages`

如果以上命令报错的话，我们还可以通过一个 --distribute 选项让 virtualenv 使用新的基于发行版的包管理系统而不是 setuptools 获得的包。  –distribute 选项会自动在新的虚拟环境中安装 pip ，这样就不需要手动安装了。

`virtualenv --distribute nowamagic_venv`

接着，我们可以进行虚拟环境的激活，在终端中输入：

`$ . venv/bin/activate`

或者

`$ source $ENV_BASE_DIR/$ENVIRONMENT_NAME/bin/activate`

即可进入虚拟环境。

当然要退出环境也很简单，在终端之中输入：

`deactivate`

即可成功退出。

### 2. 通过conda命令来创建虚拟环境
如果你安装了 Anaconda 或者 miniconda ，我们将强烈推荐你使用 conda 虚拟环境。

***查看安装环境***

- conda list 查看安装了哪些包
- conda env list 查看有哪些虚拟环境
- conda -V 查看conda的版本

***创建虚拟环境***

使用 conda 命令进行虚拟环境的创建时，必须指定需要安装的包，比如：

`conda create -n py2 python=2* anaconda`

这条命令会给我们安装 anaconda2 的版本。

** 案例1 **

安装一个名为 mydjangoapp 的虚拟环境，并安装 django

`conda create -n mydjangoapp django`

** 案例2 **

克隆一个和原系统一样的 python 环境，命名为nb：

`conda create -n nb --clone root`

** 案例3 **
当然我们也可以通过如下命令不指定具体包：

`conda create --name $ENVIRONMENT_NAME python`

** 其他 **

```
$ conda create -n py3 python=3*
$ conda create -n py2 python=2*
```
在这里，我们创建了2个 Python 版本的环境。

***环境切换***

`source activate mydjangoapp`

***退出环境***

`source deactivate`

***修改指定虚拟环境的安装包***

`conda install -n yourenvname [package]`

***移除虚拟环境***
- 移除环境中的包
`conda remove --name $ENVIRONMENT_NAME $PACKAGE_NAME`

- 移除整个环境
`conda remove -n yourenvname --all`

注意：这里的所有虚拟环境都安装在 ~Anaconda3/envs 或者 ~miniconda3/envs 目录下面.


