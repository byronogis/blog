---
title: "zx: A tool for writing better scripts"
description: 在 js 中调用 shell 命令，如同在 shell 中调用命令一样
publishedTime: 2023-11-12
tags:
  - js
  - shell
keywords: js,shell,zx,node
series: 每天认识一个 Node 包
modifiedTime: 2023-11-13

---

# zx

> [zx](https://www.npmjs.com/package/zx): A tool for writing better scripts
>
> v7.2.3
>
> 本文档仅为个人记录使用，更多内容请查看 [官方文档](https://google.github.io/zx/)
>

## 使用

编写脚本文件 `example.mjs`:

```js
$`echo hello`
```

使用下面的方式执行脚本文件

### 利用 npx 直接执行

```bash
npx zx example.mjs
```

### 全局安装后执行

#### 通过 `zx` 命令执行

```bash
npm install -g zx
zx example.mjs
```

#### 直接执行

脚本文件头部可以声明 `#!/usr/bin/env zx`，这样在全局安装的情况下，可以直接执行脚本文件(需要先修改文件为可执行文件)，而不需要使用 `zx` 命令

`example.mjs`:

```diff
+ #!/usr/bin/env zx

$`echo hello`
```

```bash
# 修改文件为可执行文件
chmod +x example.mjs
# 直接执行
./example.mjs
```

## 命令行行为

在命令行通过 zx 命令执行脚本时，有以下行为

### 无扩展名文件

如果文件没有扩展名，zx 会以扩展名为 mjs 的方式执行。

比如可以在 git 钩子中使用 zx，而不需要在文件名后面加上扩展名。

### 远程文件

如果文件名是一个 `https://`（经本地测试也可为 `http://`）开头的文件，zx 会下载文件并执行。

### 接受输入

在命令行可以直接输入内容

```bash
zx << 'EOF'
$`echo hello`
EOF
```

### 安装依赖

可通过添加参数 `--install` 在执行时自动安装缺少的依赖。（这会在当前目录下生成`node_modules` 文件夹）

```bash
zx --install << 'EOF'
import { v4 as uuid } from 'uuid'

console.log(uuid())

const {version} = await fs.readJson('./node_modules/uuid/package.json')
console.log(version)
EOF
```

并且可以在脚本文件的以来导入处指定版本号： `// @版本号`

```bash
zx --install << 'EOF'
import { v4 as uuid } from 'uuid' // @^8

console.log(uuid())

const {version} = await fs.readJson('./node_modules/uuid/package.json')
console.log(version)
EOF
```

## API

### $`command`

> [Tagged templates](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#tagged_templates), 一种利用模板字符串调用函数的方式

```bash
zx << 'EOF'
$`echo hello`
$`pwd`
$`git --version`
EOF
```

### cd()

切换当前工作目录。

只更改 `process.cwd()`，不会更改 `$.cwd`。


```js
zx << 'EOF'
$`pwd`
cd('..')
$`pwd`
EOF
```

### echo()

输出内容

```js
zx << 'EOF'
echo('hello')
EOF
```


### 一些工具依赖包可以直接使用

- [globby](https://www.npmjs.com/package/globby)
- [which](https://www.npmjs.com/package/which)
- [minimist](https://www.npmjs.com/package/minimist)
- [chalk](https://www.npmjs.com/package/chalk)
- [fs-extra](https://www.npmjs.com/package/fs-extra)
- [node:os](https://nodejs.org/api/os.html)
- [node:path](https://nodejs.org/api/path.html)
- [yaml](https://www.npmjs.com/package/yaml)

## 配置

### $.cwd

当前工作目录。

未指定时，zx 使用 `process.cwd()` 作为当前工作目录；当指定了 `$.cwd` 时，zx 会使用 `$.cwd` 作为当前工作目录。

并且由于 `cd()` 只会改变 `process.cwd()`，所以，在指定后，每次进行新的操作时工作目录都会是指定的目录。


```bash
zx << 'EOF'
$.cwd = process.cwd()
echo($.cwd)
EOF
```

## FAQ

### 配置 `$.cwd` 后，为什么 `cd()` 无效？

因为 `cd()` 只会改变 `process.cwd()`，所以，在指定后，每次进行新的操作时工作目录都会是指定的工作目录。

但是可以通过 `cd ../ && pwd` 类似的方式来临时切换工作目录。

```bash
zx << 'EOF'
$.cwd = process.cwd()
await $`pwd`
cd('../')
await $`pwd`
await $`cd ../ && pwd`
EOF
```
