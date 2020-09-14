## 从0开始开发一个node-cli工具

创建一个 test-cli 目录，并进入该目录

```shell
$ mkdir test-cli && cd test-cli
```

初始化npm

```shell
$ npm init -y
```

新建一个index.js

```js
#!/usr/bin/env node

console.log('Hello cli')
```

通过`package.json`的`bin`来指定对应的脚本位置，key为执行cli的命令:

```json
"bin":{
  "test-cli": "./index.js"
}
```

### 本地测试

#### 本地包的安装及卸载

以下命令的执行都需要在项目根目录下

- Yarn
  - 安装：`yarn link` 或 `yarn global add file:$(echo $(pwd))`
  - 卸载：`yarn unlink` 或 `yan global remove test-cli`
- Npm
  - 安装：`npm link` 或 `npm install -g`
  - 卸载：`npm unlink` 或 `npm uninstall -g test-cli`

在 Terminal 中通过 `test-cli` (也就是 `package.json` 中 `bin` 指定的key) 执行来测试我们的cli工具

### 其他

在实际项目开发中，出于可维护性考虑，我们的cli项目可能会拆分多个文件。但在发布时，为了减小发布包的大小，需要打包成一个文件，此时可以借助 [@vercel/ncc](https://www.npmjs.com/package/@vercel/ncc) 这个库来完成。该仓库不需要做任何配置，同时也可以支持 TS 项目的打包。

