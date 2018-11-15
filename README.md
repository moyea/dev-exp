# 前端开发中的一些经验
## 微信开发中一些问题解决

#### 微信ios中点击链接或图片出现黑色zhe遮罩层解决方案
```css
*{
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
}
```

#### 取消微信浏览器下拉/上拉显示黑色背景
```css
html{
  position: fixed;
}
```
注意：此种方案可能导致滚动不流畅的情况发生

#### 微信中滚动不流畅解决方案
```css
html{
   -webkit-overflow-scrolling: touch;
}
```

#### 如何使用命令行打开webstorm

打开WebStorm,  Main Menus > Tools > Create Command-Line Launcher...  确定
默认会创建 /usr/bin/webstorm 
之后可以在命令行中打开WebStorm
```bash
$> webstorm
```
也可以 
```
$> webstorm <文件名/目录名> 
```
打开文件或目录

