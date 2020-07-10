我们有一些私有包，比如wxb-logs:日志记录、wxb-express-security:安全限制等包，不能发到外网，之前我们采用
```
    "serve-favicon": "~2.4.2", // 公有包
    "wxb-express-security": "git+http://gitlab.source3g.com:10080/npm/wxb-express-security#semver:^1.0.0",
    "wxb-logs": "git+http://gitlab.source3g.com:10080/npm/wxb-logs.git#semver:^1.0.0"
```
这种方式来安装包，和公有包比起来，安装要麻烦一些，期望能像公有包一样来安装，而且还能保证相对的安全。
```
    "serve-favicon": "~2.4.2", // 公有包
    "wxb-express-security": "^1.0.0", // 私有包
    "wxb-logs": "^1.0.0"  // 私有包
```
npm私有化的作用

1、私密性

2、统一管理企业内的业务组件或者模块

3、npm更快速、稳定、安全

使用方式有两种方式

# 一、npm 
设置
```
npm config set registry http://nexus3.source3g.com/repository/wxbnpm/
```
查看 

```
npm config get registry
```
![avatar](img/11.png)


# 二、nrm (建议使用)
添加源

```
nrm add wxbnpm http://nexus3.source3g.com/repository/wxbnpm/
```
使用源
```
nrm use wxbnpm
```
![avatar](img/12.png)


带 * 的是当前源