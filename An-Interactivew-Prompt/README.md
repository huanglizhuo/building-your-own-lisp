#交互式提示

#读入，求值，输出

  既然我们需要构建编程语言，我们就得通过某种方式和它交互。才C得编译器中，你可以修改你的程序并重新编译它。如果我们可以语言互动那么将是一个进步，这样就可以很快看到不同情形时的结果了。这样我们就得构建一种叫交互式提示的东西。

  这是一个提示用户给一些输入，然后给出一些信息。这是测试编程语言结果最快也是最简单的方式。这种系统被称之为REPL(交互式编程环境)，REPL是 read-evaluate-print-loop 的缩写。如果你以前用过 python 的话，就知道它是于编程语言交互的一种常见方式。

  在构建完整地REPL之前，我们先从简单地开始。先构建一个不论用户输入什么都直接输出。等把这个完成了，接下来我们就可以像lisp那样解析用户的输入并求值了。

![reptitle sort like REPL](./reptile.png)

#交互式提示

  基本的设置是写一个循环不断读入我们的信息，然后继续等待输入。可以用`fgets`获取用户的输入，这是一个C的函数，它会在新行之前一直读入。同时需要一个存储输入的地方。这样需要声明一个固定的输入缓冲区。

  一旦我们把用户的输入存储下来了，随后就用`printf`可以把它打印给用户了。

  ```c
  #include<stdio.h>
  int main(int argc,char** argv){
    //打印版本和退出方式
    puts("lisp version 0.0.0.0.1");
    puts("Press Ctrl_c to Exit\n");
    //一个死循环
    while(1){
        //输出我们的提示
        fputs("lispy> ",stdout);
        //读入用户输入的一行 最大为2048
        fputs(input, 2048 stdin);
        //打印出用户的输入
        printf("No you're a %s",input);
    }
    return 0;
  }
  ```

  上面代码中得`//`是C语言中的注释，后面接得语句是不会被执行的。

  下面我们深入解释下这段程序。

  `static char input[2048]`这行声明了一个大小事2048的字符数组。这里存了一些在整个程序中都可以访问的数据。这个数组将会用来存储命令行的输入。关键词`static`使变量只能在该文件中使用，而`[2048]`则声明了大小。

  通过`while(1)`定义了一个死循环。也就是说在这个循环中的代码会一直运行。

  用`fputs`函数输出提示。这与`puts`函数有一点区别，`puts`函数不会追加新行。用`fgets`函数获得命令行的输入。这俩个函数都需要一个文件来供读写，这里我们用特殊的变量`stdin`,`stdout`来代替文件。它们都是在`<stdio.h>`中声明特殊变量，分别代表从命令行的输入和输出。当把变量传给`fgets`函数时，这个函数会等待用户输入一行文字，当输入没结束之前它会把输入都存在`input`缓冲区中，包括换行字符。即使`fgets`函数不会读太多的数据但我们仍然需要提供一个`2048`大小的缓冲区。

  用`printf`函数将信息返回给用户。这是一个可以打印很多元素的函数。它可以有参数中提供的样式打印字符。比如这个例子中的`%s`，它会把后面的参数当做字符串输出。

  想知道更多地格式控制信息请参看[printf](http://en.cppreference.com/w/c/io/printf)文档。

  我怎么能知道向`fgets``printf`这样的函数？

  知道这些标准函数和怎么使用它们以及什么时候使用它们不是很快就知道的。当面对问题时，需要经验来指导你怎么通过库函数解决问题。幸运的是C得标准库很小并且所有的函数都可以通过练习来学习。如果你想学习写基本的库，那么很有必要参看[知道文档](http://en.cppreference.com/w/c)。

#编译

  你可以使用下面的命令编译你的代码

  >cc -std=c99 -Wall prompt.c -o prompt

  编译完成后应该尝试运行它。`Ctrl+c`可以退出程序的运行。如果一切正确你的程序应该是这样的：

  ```c
  Lispy Version 0.0.0.0.1
  Press Ctrl+c to Exit

  lispy> hello
  No You're a hello
  lispy> my name is Dan
  No You're a my name is Dan
  lispy> Stop being so rude!
  No You're a Stop being so rude!
  lispy>
  ```

#编辑输入

  如果你用是 Linux 或者 Mac 你可能注意到当你使用方向键编辑你的输入时会有些奇怪的事情发生。

  ```c
Lispy Version 0.0.0.0.3
Press Ctrl+c to Exit
  
lispy> hel^[[D^[[C]
  ```

  使用方向键输入了好多奇怪的字符，而不是在输入字符间移动光标位置。我们想要做的是可以在行内移动光标编辑和修改我们的输入。

  在 Windows 上是没有问题的。而在 Linux 或者 Mac 上是有一个叫`editline`的库来提供支持的。在 Linux或者 Mac上我们需要用`editline`提供的函数替换`fgets``fputs`这样的函数。

  如果你是在Windows上开发的那么可以跳过该节。

#使用Editline

  在`editline`库中提供了两个叫`readline`和`add_history`的函数。`readline`函数是用来读取输入的。`add_history`函数允许我们记录输入这样就可以使用方向键回顾之前的输入行。

  在代码中替换`fgets``fputs`函数。

  ```c
  #include<stdio.h>
  #include<stdlib.h>

  #include<editline/readline.h>
  #include<editline/history.h>
  int main(int argc, char** argv) {
   
  /* Print Version and Exit Information */
  puts("Lispy Version 0.0.0.0.1");
  puts("Press Ctrl+c to Exit\n");
   
  /* In a never ending loop */
  while (1) {
    
    /* Output our prompt and get input */
    char* input = readline("lispy> ");
    
    /* Add input to history */
    add_history(input);
    
    /* Echo input back to user */    
    printf("No you're a %s\n", input);

    /* Free retrieved input */
    free(input);
    
    }
  
  return 0;
  }
```

  这次我们引入了新的头文件。`#include<stdio.h>`文件提供了`free`函数。`#include<edtline/readline.h>`和`#include<editline/history.h>`提供了`editline`，`readline`，`add_history`函数。

#带Editline的编译

  如果你立刻尝试使用这段代码来编译那么会有错误。这是因为需要你的电脑安装了`editline`库。

  在Mac上`editline`库来自命令行工具。这部分的安转在第二章有。然而你坑仍然会由错误产生，会告诉你history头文件没有发现。把`#include<editline/huistory.h>`删除。

  在Linux上你可以通过`sudo apt-get install libedit-dev`安装。在Fedora上使用`su -c "yum install libedit-dev*"`
  
  这些做完后再次尝试编译发现这回错误是：

  >undefined reference to 'readline'

  >undefined reference to 'add_history'

  这意味着没有把`editline`和你的程序连接起来。这时可以通过给编译命令加上`-ledit`标志告诉编译器要链接`editline`

  >cc -std=c99 -Wall test.c -ledit -o test

  运行你的代码现在你的程序就可以编辑你的输入了。

  ```
    还是不能运行！

    有些系统安装，链接`editline`方式可能有些不同，比如Arch linux 对应的库名叫`histedit.h`.如果如果有问题可以上网查找相应地安装方案。
  ```

#C预处理器

  这样小的工程对于不同的操作系统编写不同的代码还行，但如果我想把我的代码发给朋友让他在不同的操作系统上帮我一下，那么就会产生很多问题。最理想的就是我的代码写完后可以在任何系统，任何计算机上运行。这对C来说是个普遍的问题，我们叫它可移植性。然而并没有简单地方法解决这个问题。但C提供了叫做预处理器的机制解决这个问题

  预处理器是一个可以在编译之前运行的程序。它有很多用途，很多时候我们并没有意识到我们在使用它。任何以`#`开头的行都是一个预处理命令。我们在*include*头文件时，预处理器让我们可以访问标准库和其它库。

  另一个应用就是检测代码是在哪个操作系统上编译的，并由此触发不同的代码。

  这也是我们想要用的方式。如果你在 wimdows 上那么就使用有 `readline` `add_history` 函数的代码，否则就用引入了`editline`头文件的代码。

  声明编译器需要调用怎样的代码，我们用 `#ifdef` ,`#else` 以及 `#endif` 这样的预处理语句。它们和`if`的作用相当。这样就可以让我们的代码在不同的操作系统上编译了。

  ```c
  #include <stdio.h>
#include <stdlib.h>

/* If we are compiling on Windows compile these functions */
#ifdef _WIN32
#include <string.h>

static char buffer[2048];

/* Fake readline function */
char* readline(char* prompt) {
  fputs(prompt, stdout);
  fgets(buffer, 2048, stdin);
  char* cpy = malloc(strlen(buffer)+1);
  strcpy(cpy, buffer);
  cpy[strlen(cpy)-1] = '\0';
  return cpy;
}

/* Fake add_history function */
void add_history(char* unused) {}

/* Otherwise include the editline headers */
#else
#include <editline/readline.h>
#include <editline/history.h>
#endif

int main(int argc, char** argv) {
   
  puts("Lispy Version 0.0.0.0.1");
  puts("Press Ctrl+c to Exit\n");
   
  while (1) {
    
    /* Now in either case readline will be correctly defined */
    char* input = readline("lispy> ");
    add_history(input);

    printf("No you're a %s\n", input);
    free(input);
    
  }
  
  return 0;
}
  ```
#参考

prompt_unix.c

```c
#include <stdio.h>
#include <stdlib.h>

#include <editline/readline.h>
#include <editline/history.h>

int main(int argc, char** argv) {
   
  /* Print Version and Exit Information */
  puts("Lispy Version 0.0.0.0.1");
  puts("Press Ctrl+c to Exit\n");
   
  /* In a never ending loop */
  while (1) {
    
    /* Output our prompt and get input */
    char* input = readline("lispy> ");
    
    /* Add input to history */
    add_history(input);
    
    /* Echo input back to user */    
    printf("No you're a %s\n", input);

    /* Free retrived input */
    free(input);
    
  }
  
  return 0;
}
```

prompt_windows.c

```c
#include <stdio.h>

/* Declare a buffer for user input of size 2048 */
static char input[2048];

int main(int argc, char** argv) {

  /* Print Version and Exit Information */
  puts("Lispy Version 0.0.0.0.1");
  puts("Press Ctrl+c to Exit\n");

  /* In a never ending loop */
  while (1) {

    /* Output our prompt */
    fputs("lispy> ", stdout);

    /* Read a line of user input of maximum size 2048 */
    fgets(input, 2048, stdin);

    /* Echo input back to user */
    printf("No you're a %s", input);
  }

  return 0;
}
```

prompt.c

```c
#include <stdio.h>
#include <stdlib.h>

/* If we are compiling on Windows compile these functions */
#ifdef _WIN32
#include <string.h>

static char buffer[2048];

/* Fake readline function */
char* readline(char* prompt) {
  fputs(prompt, stdout);
  fgets(buffer, 2048, stdin);
  char* cpy = malloc(strlen(buffer)+1);
  strcpy(cpy, buffer);
  cpy[strlen(cpy)-1] = '\0';
  return cpy;
}

/* Fake add_history function */
void add_history(char* unused) {}

/* Otherwise include the editline headers */
#else
#include <editline/readline.h>
#include <editline/history.h>
#endif

int main(int argc, char** argv) {
   
  puts("Lispy Version 0.0.0.0.1");
  puts("Press Ctrl+c to Exit\n");
   
  while (1) {
    
    /* Now in either case readline will be correctly defined */
    char* input = readline("lispy> ");
    add_history(input);

    printf("No you're a %s\n", input);
    free(input);
    
  }
  
  return 0;
}
```