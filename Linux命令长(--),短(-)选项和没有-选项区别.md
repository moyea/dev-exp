# Linux命令长(`--`),短(`-`)选项和没有`-`选项区别

https://blog.csdn.net/wq6ylg08/article/details/88812451

https://unix.stackexchange.com/questions/96703/what-is-a-non-option-argument

> 在类Unix系统中, ASCII 连字符减号开始选项; 新的 (和 [GNU](https://en.wikipedia.org/wiki/GNU)) 约定用两个连字符然后接一个单词 (e.g. `--create`) 来表示选项的使用，而旧约定 (仍可作为常用选项使用) 使用一个连字符然后使用一个字母(e.g., `-c`); 如果一个连字符后跟两个或多个字母，则可能意味着已指定了两个选项，或者可能意味着第二个及后续字母是第一个选项的参数(e.g. , 文件名或日期) .
>
> 两个不带字母的 (`--`) 的连字符减号可能表示不应将其余参数视为选项，例如，如果文件名本身已连字符开头，或其他参数用于内部命令(e.g., [sudo](https://en.wikipedia.org/wiki/Sudo)). 在使用更具描述性的选项名称情况下，有时还会使用双连字符减号作为长选项的前缀. 这是GNU软件的常用功能. *[getopt](https://en.wikipedia.org/wiki/Getopt)*方法和程序,以及 *[getopts](https://en.wikipedia.org/wiki/Getopts)* 通常用于解析命令行选项。
>
> Unix 命令名称, 参数和选项区分大小写 (除了少数示例，主要是来自其他操作系统的流行命令已移植到Unix的示例).
>
> -- [维基百科](https://en.wikipedia.org/wiki/Command-line_interface#Option_conventions_in_Unix-like_systems)

## 一. Linux命令长选项`--`，短选项`-`和没有`-`选项背景:

在解释这些区别之前需要先了解一下有关linux的背景知识，这样大家对这些区别及linux也会有更深入的了解：

1. Unix操作系统在操作风格上主要分为System V和BSD(目前一般采用BSD的第4个版本SVR4)，前者的代表的操作系统有Solaris操作系统，在Solaris1.X之前，Solaris采用的是BSD风格，2.x之后才投奔System V阵营。后者的代表的操作系统有FreeBSD。
   - System V它最初由AT&T开发，曾经也被称为AT&T System V，是Unix操作系统众多版本中的一支。在1983年第一次发布，一共发行了4个System V的主要版本，System V Release4，或者称为SVR4，是最成功的版本，该版本有些风格成为一些UNIX共同特性的源头，如下表格的初始化脚本/etc/init.d。用来控制系统的启动和关闭。
   - BSD（Berkeley Software Distribution，伯克利软件套件）是Unix的衍生系统，1970年代由伯克利加州大学（Uni Versity of California, Berkeley）开创。BSD用来代表由此派生出的各种套件集合。
2. System V和BSD风格以及他们与Linux的关系：
   - System V 和BSD同出于AT＆T实验室的两个不同的部门，SystemV是一个Unix的商业化标准，BSD为Unix标准化的Berkeley风格。
   - 由于Linux是Linus Torvalds在以Unix为构架的系统上重新开发的，但仍沿用了两大Unix系统进程的风格，实事上应该确切的说Linus Torvalds只开发了kernel，而软件依然来自GNU和GPL两个组织。

目前只有Slackware是Linux发行版中唯一使用BSD风格的版本。其他的就是FreeBSD、NetBSD和OpenBSD三个著名的BSD发行版，并遵循「GPL规范」。在商业版的Unix及多数Linux发行版使用SystemV风格的init『可能有版权纠纷问题』。Linux代表的有：RedHat、Suse、MDV、MagicLinux、Debian等几乎大部分发行版。Unix代表的有AIX、IRIX、Solars、HP-UX。

**参数前有`-`的是System V风格，参数前没有`-`的是BSD风格**

