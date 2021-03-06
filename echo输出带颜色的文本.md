# echo输出带颜色的文本

## 格式

```
\033[显示方式;前景色;背景色;m
```

linux终端下输出带颜色的文字，只需要在输出文字前添加以上格式的代码。其中\033是ESC键的八进制，`\033[`即告诉终端后面是设置颜色的参数，其中显示方式，前景色，背景色均是数字。

## 参数含义

| 显示方式 | 释义         |
| -------- | ------------ |
| 0        | 终端默认设置 |
| 1        | 高亮显示     |
| 4        | 使用下划线   |
| 5        | 闪烁         |
| 7        | 反白显示     |
| 8        | 不可见       |

| 前景色 | 背景色 | 颜色   |
| ------ | ------ | ------ |
| 30     | 40     | 黑色   |
| 31     | 41     | 红色   |
| 32     | 42     | 绿色   |
| 33     | 43     | 黄色   |
| 34     | 44     | 蓝色   |
| 35     | 45     | 紫红色 |
| 36     | 46     | 青蓝色 |
| 37     | 47     | 白色   |

## 示例

使用时可将所有控制参数都用上，也可以只使用前景色或背景色。

但使用时需要注意，如果输出带颜色的字符后没有恢复终端默认设置，后续命令的的输出仍旧会采用之前的颜色。因此如果不希望影响后续文字的输出，需在输出带颜色的文字之后恢复终端默认设置，如果只是简单设置文字颜色，推荐如下方式：

```
echo "\033[31m红色文字\033[0m"
echo "\033[32m绿色文字\033[0m"
echo "\033[5;33m白色闪烁文字\033[0m"
```



