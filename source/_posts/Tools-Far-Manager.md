---
title: Far Manager 配置
toc: true
permalink: /posts/Tools-FarManager/
date: 2022-03-10 11:10:11
categories: 工具
tags: 工具
---



## Install

对于 **Scoop** 用户

```sh
scoop install far
```

**msi** 安装

[**Far Manager v3.0 build 5959 x64** (2022-02-02)](https://www.farmanager.com/files/Far30b5959.x64.20220202.msi)

## Setup

### 调节字体

在底部命令行输入 

```sh
chcp 65001
```

![设置字体为Consolas](https://gitee.com/relifes/image/raw/master/img/20220310112359.png)

设置完等宽字体后，可能会出现内容显示不满整个屏幕的情况，这个时候重新缩小窗口再放大即可

<!--more-->

### 常用操作按键

**文件操作**

- 切换面板 - `tab`
- 新建文件夹 - `F7`
- 新建文件 - `Shift + F4`
- 编辑文件 - `F4`
- 移动文件 - `F6`
- 重命名文件 - `Shift + F6`
- 删除文件 - `F8`
- 压缩与解压缩文件 - `Shift F1 / F2` 可以使用Shift与方向键选中多个文件
- 切换至命令行 - `Ctrl + O`

**编辑器操作**

- 保存文件 - `F2`
- 退出编辑 - `F4`
- 删除一行 - `Ctrl + y`
- 选中一行 - `Shift + End / Home`
- 将选中的块左右移动 - `Ctrl u/i`
- 以word为单位跳动光标 - `Ctrl + left / right`
- 隐藏或显示命令行 - `Ctrl + B` 当快捷键熟练后使用
- 隐藏或显示顶部信息 - `Ctrl + Shift + B`
- 替换 - `Ctrl + F7`
- 查找 - `F7`
- 切换至其他文件或面板 - `F12`
- 修改页面编码 - `Shift + F8` 一般默认为 GBK

### Opinions中各项Setting设置

<img src="https://gitee.com/relifes/image/raw/master/img/20220310122202.png" style="zoom:67%;" />

- System Setting

<img src="https://gitee.com/relifes/image/raw/master/img/20220310132856.png" style="zoom:67%;" />

勾选上Auto save setup，手动保存设置快捷键为`Shift + F9`

- Panel Setting

<img src="https://gitee.com/relifes/image/raw/master/img/20220310133136.png" alt="image-20220310133136878" style="zoom: 80%;" />

- Editer Setting

<img src="https://gitee.com/relifes/image/raw/master/img/20220310133825.png" alt="image-20220310133825029" style="zoom:67%;" />

在这个选项里设置tab size为4个空格，并打开Auto indent实现换行自动缩进

其他设置如果有需要可以自行设置

### 设置C++编译选项

在`F9 - Commands - File associations`中，按`Ins`新建配置，

<img src="https://gitee.com/relifes/image/raw/master/img/20220310134909.png" alt="image-20220310134909804" style="zoom: 80%;" />

<img src="https://gitee.com/relifes/image/raw/master/img/20220310134939.png" alt="image-20220310134939879" style="zoom:80%;" />

```sh
g++ -O2 -std=gnu++14 -Wall -Wextra -Wfatal-errors -Wl,--stack=256000000 -o !.exe !.cpp -DLOCAL_DEFINE
```

这行指令中

- `-O2`表示开启O2优化，对于STL的速度有显著提升
- `-std=gnu++14`指定C++的版本通常选择 `11, 14, 17`
- `-Wall`开启“所有”的警告
- `-Wextra`除-Wall外其它的警告。建议加上。
- `-Wfatal-errors`遇到第一个错误就停止，减少查找错误时间。
- `-Wl` 选项告诉编译器将后面的参数传递给链接器。
- `--stack=256000000` 指定栈空间的大小为256M
- `-DLOCAL_DEFINE` 很有用的一个参数，可以利用它来进行文件的输入输出，提交到OJ的时候也不需要注释这段代码。

```c++
#ifdef LOCAL_DEFINE
    freopen("in.txt", "rt", stdin);
    freopen("out.txt", "w", stdout);
#endif
```

### 插件设置

模板和代码补全插件 TrueTemplate

github：[chronoxor/TrueTemplate: FAR Manager editor plugin for edit and compile source files (github.com)](https://github.com/chronoxor/TrueTemplate)

Releases：[TrueTemplate-3.0.2.zip](https://github.com/chronoxor/TrueTemplate/releases/download/3.0.2/TrueTemplate-3.0.2.zip)

下载完成后，解压至Far Manager的`Plugins`目录下

然后在TrueTemplate目录下的底部命令栏输入 

```bash
load:true-tpl.x64.dll
```

![](https://gitee.com/relifes/image/raw/master/img/20220310142438.png)

此时`F11`在插件目录的最下面应该就能够看到它了

如果不行，*<u>重启解决99%问题</u>*

进入`templates - source`目录进行代码补全和模板插入的设置

![](https://gitee.com/relifes/image/raw/master/img/20220310143842.png)

进入后初始会有很多文件，我只保留了与C++相关的条目

#### true-source.xml文件

```xml-dtd
<?xml version="1.0" encoding="windows-1251"?>
<TrueTpl>
  <Include File="templates\source\true-source-c++.xml"/>
</TrueTpl>
```

这个文件是加载模板的入口文件，我只保留了C++模板的配置，这里其他配置删不删都可以

#### true-source-c++.xml文件

![](https://gitee.com/relifes/image/raw/master/img/20220310144253.png)

这个文件的作用是配置以`.cpp`结尾的插件菜单，进行**模板配置**时文件在此处定义

```xml
<!-- Шаблоны -->
<Expand Name="&amp;Templates " Init="1" To="\~Template: sol=sol"/>
<Expand Name="Template: sol" At="&AnyWhere;" SubMenu="1" To="\i'$\templates\source\sol.cpp'"/>    
```

`\i`表示外部文件，我们在source目录下新建模板文件`sol.cpp`

**注意：该文件的编码格式必须为`UTF-16LE`，否则会出现乱码错误，而我们自己用模板创建的文件可以使用默认的GBK编码**

我的模板文件配置

```c++
#include <bits/stdc++.h>
using namespace std;

#define all(x) (x).begin(), (x).end()
#define rep(i, a, b) for(int i = a; i < (b); ++i)
#define trav(a, x) for(auto& a : x)  

typedef long long ll;
typedef long double ld;
typedef pair<int, int> pii;
typedef vector<int> vi;
 
const int N = 1e5 + 10;

int main() {
    ios::sync_with_stdio(false);                                           
    cin.tie(nullptr);
    cout.precision(6);
    cout << fixed;
#ifdef LOCAL_DEFINE
    freopen("in.txt", "rt", stdin);                                              
    freopen("out.txt", "w", stdout);
#endif
    
    return 0;                                                                           
}
```

配置完成后当我们使用`Shift + F4`新建文件时，便可以加载我们的模板了

效果如下图所示：

<img src="https://gitee.com/relifes/image/raw/master/img/20220310151156.png" style="zoom:80%;" />

可以看到在文件开头引入了`true-source-c++-base.xml`文件，这是我们所需要进行自动补全配置的文件

#### true-source-c++-base.xml 文件

打开文件后可以看到，大量使用了正则表达式进行匹配，定位到`Expand`项目

![](https://gitee.com/relifes/image/raw/master/img/20220310144732.png)

`Pattern`是一个正则表达式，当满足匹配条件时，按`space`自动填充`To`后面的内容

例如 输入`if + 空格` 转变为 `if (\p) `其中 `\p` 表示光标所在位置

同时，该插件默认为我们提供了 大括号的自动匹配＋换行，但是没有中括号和小括号的自动匹配

大家可以参考我的配置自行进行更改

```xml
<Expand Name="C/C++ &amp;condition operators "                               To="\~Ternary condition=Ternary condition (?:)\~if=if ()\~else=else\~else if=else if ()\~switch=switch ()\~case=case :\~default=default:"/>
<Expand Name="Ternary condition" At="&OutWord;" SubMenu="1" Pattern="\?:"     To="\?'Condition expression:'e''?\?'True expression:'e''?\?'False expression:'e''?\b\b(\0?\1:\2)"/>
<Expand Name="test"              At="&OutWord;" SubMenu="1" Pattern="test"    To="\i'$\templates\source\test.cpp'"/>
<Expand Name="if"                At="&OutWord;" SubMenu="1" Pattern="if"      To="if (\p)"/>
<Expand Name="else"              At="&OutWord;" SubMenu="1" Pattern="els|"    To="else "/>
<Expand Name="else if"           At="&OutWord;" SubMenu="1" Pattern="ei"      To="else if (\p)"/>
<Expand                          At="&OutWord;"             Pattern="eli|"    To="else if (\p)"/>
<Expand Name="switch"            At="&OutWord;" SubMenu="1" Pattern="swi|tc"  To="switch (\p)\r{\r\r}\^case :\rdefault:\r\[\b"/>
<Expand Name="case"              At="&OutWord;" SubMenu="1" Pattern="cas|"    To="case \p:"/>
<Expand Name="default"           At="&OutWord;" SubMenu="1" Pattern="defa|ul" To="default:"/>
<!-- Операторы циклы C/C++ -->
<Expand Name="C/C++ c&amp;ycle operators "                     To="\~for=for (;;)\~forc=for (char* i\=X;*i;i++)\~fori=for (int i\=0;i&lt;X;i++)\~foru=for (unsigned i\=0;i&lt;X;i++)\~do=do while ()\~while=while ()"/>
<Expand Name="for"            At="&OutWord;" SubMenu="1" Pattern="fo|"   To="for (\p;;)"/>
<Expand Name="do"             At="&OutWord;" SubMenu="1" Pattern="do|"   To="do\r{\r\r} while (\p);"/>
<Expand Name="while"          At="&OutWord;" SubMenu="1" Pattern="whi|l" To="while (\p)"/>
<Expand Name="rep"            At="&OutWord;" SubMenu="1" Pattern="rep|"  To="rep (\p)"/> 
<Expand Name="["              At="&OutWord;" SubMenu="1" Pattern=".*\["  To="[\p]"/>
<Expand Name="first"          At="&OutWord;" SubMenu="1" Pattern=".*\.fi"  To=".first "/>
<Expand Name="second"         At="&OutWord;" SubMenu="1" Pattern=".*\.se"  To=".second "/>
<Expand Name="make_pair"      At="&OutWord;" SubMenu="1" Pattern=".*\.mp"  To=".make_pair(\p)"/>
<Expand Name="push_back"      At="&OutWord;" SubMenu="1" Pattern=".*\.pb"  To=".push_back(\p)"/>
<Expand Name="pop_back"       At="&OutWord;" SubMenu="1" Pattern=".*\.ppb"  To=".pop_back()"/>
<Expand Name="empty"          At="&OutWord;" SubMenu="1" Pattern=".*\.em"  To=".empty()"/>
<Expand Name="size"           At="&OutWord;" SubMenu="1" Pattern=".*\.sz"  To=".size()"/>
<Expand Name="("           At="&OutWord;" SubMenu="1" Pattern=".*\("  To="(\p)"/>
  
```

 可以看到我将许多平时需要#define 定义的加入进来，这样既保证了可读性，又提高了编码效率

当然`rep`这种的该 #define 还是 define

**特别需要注意的是：正则中`.*`表示全部文件，而`[`需要进行转义为`\[`，小括号同理**

如果对正则不熟悉，用我的这些配置也可

```xml
<Expand Name="test" At="&OutWord;" SubMenu="1" Pattern="test" To="\i'$\templates\source\test.cpp'"/>
```

这行配置的操作和我们设置模板时很像，都是引入一个外来文件

众所周知，竞赛中有很多模板题，我们可以在自己的代码库中建立自己的模板

那么这种方式可以让我们在文件中快速插入我们的模板，而无需打开文件复制代码

原理是一样的，Pattern 为 test 时，当我们按下空格，便会自动填充我们的模板代码

在建立模板文件时，同样需要注意文件编码为 `UTF-16 LE`

![](https://gitee.com/relifes/image/raw/master/img/20220310151737.png)

这里中文重叠的问题，目前我还没有找到很好的解决方式，不过尽量用英文命名就OK了

同时需要注意，模板文件的缩进是有问题的，第一行需要`顶格写`，其他行和第一行缩进差一个`tab`

至此我们的Far Manager的配置基本完成了

更改配色的插件叫做 Color Setup 下面是它的插件链接，不过目前还是有很多bug，大家可以自行探索 

[Far PlugRing - plugin information (farmanager.com)](https://plugring.farmanager.com/plugin.php?pid=930&l=en))

Far Manager还有很多有用的功能（宏功能）和插件，不过设置完成这些后，对于打算法竞赛是足够了，如果有更多需求，请参考官方文档和论坛进行探索



== END ==

