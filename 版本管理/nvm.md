 nvm全名node.js version management，顾名思义是一个nodejs的版本管理工具。通过它可以安装和切换不同版本的nodejs。下面列出下载、安装及使用方法。


# 一、安装

## linux安装nvm
```
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
```

> 地址可以找最新的地址

### 执行

```
source ~/.bash_profile
```

## window下安装

可在点此在[github](https://github.com/coreybutler/nvm-windows/releases)上下载最新版本,本次下载安装的是windows版本。打开网址我们可以看到有两个版本：

nvm-setup.zip：安装版，推荐使用

# 二、nvm常用命令

## 查看nvm 版本

```
nvm --version
```

## 查看node所有版本

```
nvm ls-remote
```

## 切换nodejs版本

```
nvm use 8.9.4
```

## 安装nodejs版本


### 安装10.15.3
```
nvm install 10.15.3
```


## 三、其它命令
### 查看npm registry

```
npm get registry
```
### npm淘宝镜像

```
npm config set registry http://registry.npm.taobao.org/
```

### yarn安装
```
npm install yarn -g
```

### 查看yarn registry
```
yarn config get registry
```
### yarn淘宝镜像

```
yarn config set registry http://registry.npm.taobao.org/
```

## nodejs常用版本

```
8.9.4
10.15.3
11.10.1
12.13.0
```

ps: 用nvm 安装node会自带npm，如果没有npm, 把当前版本删除了再安装

切换源可以用nrm来切换，[参照文档](nrm.md)
