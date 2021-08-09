# 使用nrm管理源

<font color='red'>nrm</font>是一个NPM管理器，可以使用<font color='red'>nrm</font>在不同的源切换。

## 安装nrm

```bash
npm install -g nrm
```

## 列出可选的源

```bash
nrm ls

# 结果，带* 的是当前使用的源。
* npm -------- https://registry.npmjs.org/
  yarn ------- https://registry.yarnpkg.com/
  cnpm ------- http://r.cnpmjs.org/
  taobao ----- https://registry.npm.taobao.org/
  nj --------- https://registry.nodejitsu.com/
  npmMirror -- https://skimdb.npmjs.com/registry/
  edunpm ----- http://registry.enpmjs.org/
```

## 切换源

```bash
# 切换到taobao
nrm use taobao
```

## 增加源

```bash
nrm add  <registry> <url> [home]
```

## 删除源

```bash
nrm del <registry>
```

## 测试速度

还可以通过 `nrm test `测试相应源的响应时间

```bash
nrm test npm 
```

