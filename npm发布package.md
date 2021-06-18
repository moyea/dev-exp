# NPM发布package操作

## 创建npm user
- 创建用户账户

  为了发布包，需要一个npm的账户。如果没有账户，可以通过**npm adduser**来创建账户或者在[https://www.npmjs.com/signup](https://www.npmjs.com/signup)创建用户，之后使用**npm login**来操作你的账户。

- 测试账户是否有效

	1. 在命令行输入**npm whoami**\(这只表明账户存储到了本地\)

	2. 确认账户已经被添加到了[**https://www.npmjs.com/~\<username>**](https://www.npmjs.com/~<username>)，如: https://www.npmjs.com/~jerry

## 准备发布内容

- 查看内容

  除了本地**.gitignore**或**.npmignore**指明了的目录及文件，发布的package中将包含目录中所有的内容。

- 查看package.json

  根据[使用package.json](https://docs.npmjs.com/getting-started/using-a-package.json)确保能反映你包中的内容。

- 确定包名的唯一性

  注意名字必须唯一，不能与已有的package冲突，否则在发布时会导致403错误

- 使用文档\(readme.md\)

  可以通过该文件来描述你的package具体作用及使用方式等。发布时，此文档将显示在下载页面上。可以参考其他package的readme。

## 发布

在命令行执行 `npm publish` 命令。之后访问**https//npmjs.com/package/&lt;package&gt;**。若能看到你发布包的页面，就证明发布成功了。

## 更新

### 版本更新

通过以下命令

```bash
npm version <update_type>
```

或者直接修改package.json去更新对应的主要、次要及补丁版本信息。

之后再次执行:

```bash
npm publish
```

测试: 访问 https://npmjs.com/package/\<package>。查看版本号是否被更新。

### 文档更新

除非发布新版本的package，否则不会更新网站上显示的描述文件，所以可以运行

```
npm version patch
npm publish
```

去更新网站上显示的文档。

## 撤销发布

### 撤销单个版本

```shell
npm unpublish [<@scope>/]<pkg>@<version>
```

### 撤销整个包

```
npm unpublish [<@scope>/]<pkg> --force
```

注: 取消发布需要注意 [npm取消发布协议](https://www.npmjs.com/policies/unpublish)

