# Git commit message规范

## 零、前言

出于使团队在git上清晰地看到每次提交是什么类型的内容，以及便于追踪提交记录的目的，特新建该规范。

以下是规范的内容和在项目中配置commit message格式验证的脚本：

## 一、Message格式

```
<type>(<scope>): <summary>
// 空行
<body>
// 空行
<footer>
```

## 二、描述

```
# Commit Message Header
# ---------------------
#
# <type>(<scope>): <short summary>
#   │       │             │
#   │       │             └─⫸ 中文简短描述 | 一般现在时小写字母开头的简短描述.
#   │				│									 中英文末尾均不需要句号.
#   │       │
#   │       └─⫸ Commit Scope: *|xxx.ts|utils|XSA-8888| none
#   │
#   └─⫸ Commit Type: feat|fix|docs|style|refactor|test|chore
#
#
# Commit Message Body
# ---------------------
# 
# <body>
# 每行不超过100个字符，主要回答一下几个问题:
# * 为什么有必要进行此次更改?
# * 怎样会导致该问题出现？
# * 是否会带来其他副作用?
#
#
# Commit Message Footer
# ---------------------
#
# 如果有issue链接，可以在此处列出
# BREAKING CHANGE - 不兼容的变动及迁移方式
#
# ```
# BREAKING CHANGE: <breaking change summary>
# <BLANK LINE>
# <breaking change description + migration instructions>
# <BLANK LINE>
# <BLANK LINE>
# Fixes #<issue number>
# ```
#
# 不兼容的改动以 "BREAKING CHANGE: " 开头，跟随一个breaking change描述，一个空行
# 以及详细描述该breaking change 和集成方式
#
```

### 1. type取值

| type     | 说明                                         |
| -------- | -------------------------------------------- |
| feat     | 新功能(feature)                              |
| fix      | 修复问题                                     |
| docs     | 修改文档/注释等(document)                    |
| refactor | 对原提交的修改或重构（理论上不影响现有功能） |
| chore    | 修改构建工具/依赖等                          |
| test     | 测试性代码或TestCase                         |
| style    | 样式修改，皮肤更新等                         |
| revert   | 撤销操作时，git自动生成，不需要手动添加      |

### 2.注意事项

- type名称全部小写
- type后面紧跟英文冒号
- 冒号后需要保留一个空格
- (可选) scope 修改文件的范围或者TaskId（包括但不限于 doc, middleware, core, config, plugin）
- subject 少于100字，简短清晰地阐述提交内容 行末空一行
- body(可选) 补充subject，如必要性、解决的问题、可能影响的地方，可以多行 如果有链接一定附上链接
- footer(可选) bug的id、issue的id等

## 三、示例

### 示例一: Simple Refactor

```
refactor(core): 将refreshDynamicEmbeddedViews更名为refreshEmbeddedViews

增强代码可读性. 原始名称不再与该功能相匹配
# Example 3: A bug fix

```

### 示例二: Simple docs change

```
docs: 在readme.md中添加新手教程

Fixes #36332
```

### 示例三: Bug Fix

```
fix(XSA-1111): 游戏启动黑屏

由于本地数据在清理缓存时会被清理，导致部分数据无法读取

Closes XSA-1111
```

## 四、配置git提交模板

### 1. 模板文件

```
<type>(<scope>): <summary>

<body: 可选，描述本次更改的背后的动机 - 说明为什么要进行这次更改. 每行不超过100个字符. >

<footer: 可选，用于描述不兼容的变动及迁移方法或关闭issue>

# ────────────────────────────────────────── 100 chars ────────────────────────────────────────────┤


# Example Commit Messages
# =======================


# ─── 例子: Simple refactor ────────────────────────────────────────────────────────────────────────┤
# refactor(core): 将refreshDynamicEmbeddedViews更名为refreshEmbeddedViews
#
# 增强代码可读性. 原始名称不再与该功能相匹配
# ─────────────────────────────────────────────────────────────────────────────────────────────────┤


# ─── Example: Simple docs change ─────────────────────────────────────────────────────────────────┤
# docs: 在readme.md中添加新手教程
#
# Fixes #36332
# ─────────────────────────────────────────────────────────────────────────────────────────────────┤


# ─── Example: A bug fix ──────────────────────────────────────────────────────────────────────────┤
# fix(XSA-1111): 游戏启动黑屏
#
# 由于本地数据在清理缓存时会被清理，导致部分数据无法读取
#
# Closes XSA-1111
# ─────────────────────────────────────────────────────────────────────────────────────────────────┤


# Laix Game Commit Message Format
# ===============================
#
# 完整的 Laix Game Commit Message Format 可以访问以下链接:
# https://wiki.liulishuo.work/pages/viewpage.action?pageId=187867250
#
# 以下包含一些最常用的信息.
#
# 每个commit message 包括一个 *header*, 一个 *body*, 和一个 *footer*.
#
# <header>
# <BLANK LINE>
# <body>
# <BLANK LINE>
# <footer>
#
# header部分是必须的.
# body部分与footer部分是可选的.
# commit message中任意一行不能超过100个字符
#
#
# Commit Message Header
# ---------------------
#
# <type>(<scope>): <short summary>
#   │       │             │
#   │       │             └─⫸ 中文简短描述 | 一般现在时小写字母开头的简短描述. 中英文末尾均不需要句号.
#   │       │
#   │       └─⫸ Commit Scope: *|xxx.ts|utils|XSA-8888| none
#   │
#   └─⫸ Commit Type: feat|fix|docs|style|refactor|test|chore
#
#
# Commit Message Body
# ---------------------
#
# 每行不超过100个字符，主要回答一下几个问题:
# * 为什么有必要进行此次更改?
# * 怎样会导致该问题出现？
# * 是否会带来其他副作用?
#
# Commit Message Footer
# ---------------------
#
# 如果有issue链接，可以在此处列出
# BREAKING CHANGE - 不兼容的变动及迁移方式
#
# ```
# BREAKING CHANGE: <breaking change summary>
# <BLANK LINE>
# <breaking change description + migration instructions>
# <BLANK LINE>
# <BLANK LINE>
# Fixes #<issue number>
# ```
#
# 不兼容的改动以 "BREAKING CHANGE: " 开头，跟随一个breaking change描述，一个空行
# 以及详细描述该breaking change 和集成方式
#
```

### 2.全局配置

git添加以下配置，以忽略`#`开头的注释行

```
git config --global --add commit.cleanup strip
```

这样提交时修改以下行即可

```
<type>(<scope>): <summary>

<body: 可选，描述本次更改的背后的动机 - 说明为什么要进行这次更改. 每行不超过100个字符. >

<footer: 可选，用于描述不兼容的变动及迁移方法或关闭issue>
```

### 3.SourceTree中使用

在“SourceTree偏好设置 → 提交” 的文本编辑框中，粘贴模板内容：

同时勾选以下两项：

- 使用等宽字体显示评论信息
- 在输入提交信息窗口中显示换行提示竖线，且允许长度为100

注意：在粘贴时可能导致SourceTree卡死或者习惯使用命令行提交，可通过以下方式配置git全局模板(在SourceTree中编辑和直接修改文件是一样的)：

#### 1）新建模板文件

```
vi ~/.git_commit_msg_template
```

#### 2)按 i 进入编辑模式，粘贴模板内容，按 esc 退出编辑模式，输入 :wq 保存退出

#### 3)更新git配置

```
git config --global commit.template ~/.git_commit_msg_template
```

## 五、在项目中配置commit msg格式验证

### 1. 安装验证工具

```
yarn add -D validate-commit-msg husky
```

### 2. 在.vcmrc中指定或者在package.json中指定

#### 1)在.vcmrc文件中指定,该文件需要是一个有效的json文件

```
{
  "types": ["feat", "fix", "docs", "style", "refactor", "test", "chore", "revert"],
  "scope": {
    "required": false,
    "allowed": ["*"],
    "validate": false,
    "multiple": true
  },
  "warnOnFail": false,
  "maxSubjectLength": 100,
  "subjectPattern": ".+",
  "subjectPatternErrorMsg": "请输入commit message!",
  "helpMessage": "Commit message 格式错误, \n请查看规范: <规范链接>",
  "autoFix": false
}
```

#### 2) 在package.json中指定

```
{
	"config":{
		"validate-commit-msg": {
			/* Your config here */
		}
	}
}
```

### 3.husky中添加commit-msg hook

```
{
	"husky": {
		"hooks": {
			"commit-msg": "validate-commit-msg"
		}
	}
}
```

