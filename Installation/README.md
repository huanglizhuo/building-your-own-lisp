#设置

  在开始用C编程志强，我们需要先安装些东西，并建立好我们需要的编程环境。由于C是很通用的语言所以这个过程会很简单。我们主要需要装来个东西。一个文本编辑器，一个编译器。

#文本编辑器

  文本编辑器是一个用来编写程序文本的程序。

  在Linux上我推荐使用gedit。不论你用的是哪个版本得linux它都是可以运行的。如果你用的是Vim或者Emacs也是可以的。请不要使用IDE。它对这样的小项目是完全不需要的，亦不会帮助你明白究竟发生了什么事情。

  在Mac上有一个叫TextWrangler的简单文本编辑器。千万不要使用Xcode。

  在Windows上推荐使用Notepad++。当然其它的也可以，千万不要使用Visual Studio。它不适合写C语言，如果你用它的话会带来很多麻烦。

#编译器

  编译器是一个将C语言源代码转换为电脑运行的二进制代码。这部分的安装取决于你用的是什么操作系统。

  编译和运行C代码需要命令行的简单使用。这部分我不会讲到，因此我假设命令行的使用你是熟悉的。如果你不会的话，请到网上搜索你操作系统对应的相关教程。

  在linux上你可以下载一些包来安装编译器。如果你在使用Ubuntu或者Debian可以通过下面命令安装

  >sudo apt-get install build-essential

  如果你用的额是Fedora或者相似的linux那么使用下面的命令

  >su -c "yum groupinsall develepment-tools"

  在Mac上你可以在Apple Store 上下载Xcode。如果你不确定怎么做你可以上网搜索"installing Xcode" 。之后你需要安装 Command Line Tools 。在Mac OSX 10.9上可以在命令行运行 
  >xcode-select --install 

  在10.9之前的版本上可以在Xcode的选项，下载,然后选择Comand Line Tools 来安装。

  在Windows上你可以下载安装安装MinGW。如果你的安装器提示你一列可选的包，记得一定要选`mingw32-base`和`msys-base`。安装完成后你要把编译器添加到你的`PATH`环境变量中。如果PATH不存在的话就创建一个，把`;C:MinGW\bin`添加到该值下。你需需要重新打开`cmd.exe`检查修改是否起效了。之后你就可以在`cmd.exe`命令行下运行编译器了。

#测试编译器

  测试你的编译器是否安装完成，请在你的命令行下运行

  >cc --version

  如果有关于编译器的相关信息返回的话那么就表示正确安装了。如果你得到的一些错误信息或者说命令无法找到，那么就是没有安装成功。你得重新来过。

#Hello World

  现在你的环境配置好了，在你可文本编辑器里输入下面的程序。最好创建一个文件夹在存放本书代码，并把该文件命名为`hello_world.c`。下面是你的C代码

  ```c
  #include<stdio.h>
  int main(int argc, char** argv){
    puts("Hello , World");
    return 0;
  }
  ```


