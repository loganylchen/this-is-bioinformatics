# python

## 虚拟环境

对于不同的开发目的，可能会用到不同的python版本，及不同的python包，避免重复的安装，可以使用虚拟环境进行操作。

### 安装

```bash
#安装
(sudo) pip install virtualenv virtualenvwrapper
```
修改~/.bash_profile或其它环境变量相关文件(如 .bashrc 或用 ZSH 之后的 .zshrc)，添加以下语句
```
export WORKON_HOME=$HOME/.virtualenvs
export PROJECT_HOME=$HOME/workspace
source /usr/local/bin/virtualenvwrapper.sh
```
修改后使之立即生效(也可以重启终端使之生效)：

```
source ~/.bash_profile
```
### 使用

**mkvirtualenv test：创建运行环境test**

**workon test: 工作在test 环境 或 从其它环境切换到test 环境**

**deactivate**: 退出终端环境

**rmvirtualenv** test：删除运行环境ENV

**mkproject** test：创建test项目和运行环境test

**mktmpenv**：创建临时运行环境

**lsvirtualenv**: 列出可用的运行环境

**lssitepackages**: 列出当前环境安装了的包

