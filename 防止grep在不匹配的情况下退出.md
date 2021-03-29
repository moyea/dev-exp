# 防止grep在不匹配的情况下退出

以下代码不会输出"after":

```bash
#!/bin/bash -e

echo "before"

echo "anything" | grep "e" # 由于grep搜索不到给定字符，会返回1

echo "after"
```

如果删除`#!/bin/bash -e`行上的`-e`选项，可以正常输出"after"，但移除该选项后就没法在脚本发生错误时就退出。

## 方式一：

如果不想将grep没有匹配项视为错误，可以在命令后接 `||`，再跟一个成功的命令。它表示命令的前一部分执行失败了，就执行`||`之后的代码。如下:

```bash
echo "anything" | grep "e" || :
```

​	注，上面的处理同时也会忽略管道中第一个命令的错误，也就是`echo "anything"`中的错误，如果不想屏蔽该错误，可以采用下面的方式进行处理，这样就只忽略grep没有匹配项的退出码

```bash
echo "anything" | { grep "e" || :;}
```

## 方式二

另一种选择是将另一条不会失败的命令添加到管道中:

```bash
echo "anything" | grep "e" | cat
```

由于`cat`是管道中的最后一条命令，所以将使用`cat`而不是`grep`的退出状态来确定管道是否失败。

## 参考链接:

[防止grep不匹配导致脚本退出](https://unix.stackexchange.com/questions/330660/prevent-grep-from-exiting-in-case-of-nomatch)

[grep退出状态码](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/grep.html#tag_20_55_14)

[ GUN Bash 内置`:`(冒号)的目的是什么?](https://stackoverflow.com/questions/3224878/what-is-the-purpose-of-the-colon-gnu-bash-builtin)

