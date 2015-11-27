#波兰表达式

  使用`mpc`我们将会实现一个酷似lisp的数学子集。它叫做[波兰表达式](http://en.wikipedia.org/wiki/Polish_notation)这是一种操作符在被操作数之前的数学表达式。

  比如...

 > 1 + 2 + 6  is  + 1 2 6
 > 6 + (2 * 9)    is  + 6 (* 2 9)
 > (10 * 2) / (4 + 2) is  / (* 10 2) (+ 4 2)

 我们需要理解描述这个表达式的语法。我们可以通过把它写出来然后再慢慢理解。

 首先，我们发现在波兰表达式中操作符总是在表达式的最前面，后面跟着一个数字或者其它的带括号的表达式。这意味着我们可以通过“语法是一个操作符后跟着一个或多个表达式，表达式是要么是一个数字或者是一个括号里包的由操作符开始的表达式”。

 下面是更严谨的表述：

 > Program    the start of input, an Operator, one or more Expression, and the end of input.
 > Expression either a Number or '(', an Operator, one or more Expression, and an ')'.
 > Operator   '+', '-', '*', or '/'.
 > Number an optional -, and one or more characters between 0 and 9

#正则表达式
  
  用我们已有的知识足够编码上述的大部分规则，但 number 和 program 可能会有些麻烦。它们包含很多我们不知道怎样表达的结构。我们不知道怎么表达输入的开始，结束，可选的字符，或者字符的范围。

  这些也是可以表达的，但需要用正则表达式。正则表达式是一种为描述简短的单词或数字的语法。使用正则表达式不可以有多重规则，但它提供了精确而且精巧的控制方式去判定哪些是有效的哪些不是。下面是一些正则表达式的基本规则。

> `.` 任何字符都可以接受
> `a` 只有字母`a`可以接受
> [abcdef] 任何在字符集`abcdef`中的字母都可以接受
> [a-f] 任何在`a`到`f`中的字符都可以接受
> `a?`  字母`a`是可选的
> `a*`  0个或多个`a`都可以接受
> `a+`  一个或多个`a`都可以接受
> `^`  输入的开头
> `$`  输入的结束

  这些是我们目前为止需要知道的规则。[这本书](http://regex.learncodethehardway.org/)写的都是关于正则的。网上还有更多关于这些规则的信息。我们会在后面的章节中使用，关于正则只需要一些基本的规则，你不需要完全掌握它。

  在`mpc`的语法中，我们用`/`分割正则表达式。使用上面的规则，我们可以用`/-?[0-9]+/`表达我们的数字规则。

#安装mpc

  在写这个语法前我们必须先包含`mpc`头文件，然后连接到`mpc`库，就像之前的`editline`一样。将你在第四章写的文件改名为`parsing.c`，在这个[仓库](http://github.com/orangeduck/mpc)下载`mpc.h`和`mpc.c`,并放入同一个文件夹下。

  在`parsing.c`顶部中加入`#include<mpc.h>`。在命令行中连接`mpc.c`。在linux上添加-lm参数来连接数学库。

  在linux 和 mac 上

> cc -stf=c99 -Wall parsing.c mpc.c -ledit -lm -o parsing

  在Windows 上

> cc -std=c99 -Wall parsing.c mpc.c -o parsing

  等等，你是不是想问为什么不是`#include<mpc.h>`

  事实上有两种方式引入C文件，一种是使用尖括号<>，一种是使用双引号""

  它们之间的不同是使用尖括号的会先搜寻系统库，双引号的先从当前目录搜寻。

#波兰式语法

  为了让上面的规则更规范，并且使用更多的正则表达式，我们可以想下面这样用波兰式写出语法的最终形式。读下面的代码，看看它和我们哪些写过的文字匹配，以及波兰式的意思。

  ```c
  
  /* Create Some Parsers */
mpc_parser_t* Number   = mpc_new("number");
mpc_parser_t* Operator = mpc_new("operator");
mpc_parser_t* Expr     = mpc_new("expr");
mpc_parser_t* Lispy    = mpc_new("lispy");

/* Define them with the following Language */
mpca_lang(MPCA_LANG_DEFAULT,
  "                                                     \
    number   : /-?[0-9]+/ ;                             \
    operator : '+' | '-' | '*' | '/' ;                  \
    expr     : <number> | '(' <operator> <expr>+ ')' ;  \
    lispy    : /^/ <operator> <expr>+ /$/ ;             \
  ",
  Number, Operator, Expr, Lispy);
  
  ```

  我们需要把它添加到第四章的代码中。把它们放在`main`函数打印版本和退出信息的前面。在程序的结尾处我们需要删除我们定义的解析器。在`main`函数返回前加入下面的代码：

  ```c
    mpc_cleanup(4,Numberm,Operator,Expr,Lispy);
  ```

#解析用户的输入

  我们的新代码创建了一个解析器用来解析波兰式语法的，但我们仍然要再用户输入时候使用它。编辑`while`循环，让它不仅仅是把用户的输入再输出出来。用下面的代码取代`printf`函数，来解析`Lispy`。

  ```c
    /* Attempt to Parse the user Input */
    mpc_result_t r;

    if (mpc_parser("<stdin>, input ,Lispy,&r")){
        mpc_ast_print(r.output)
        mpc_ast_delete(r.output)
    }else{
        mpc_err_print(r.error);
        mpc_err_delete(r.error);
    }
  ```

  `mpc_parse`函数接收`input`和`Lispy`解析器作为参数。它把解析结果复制给`r`成功返回`1`失败返回`0`。用`&`取`r`的地址。这个操作符会在后面的章节中进一步解释。

  成功后内部机构会复制给`r`。我们可以用`mpc_ast_print`打印出来，用`mpc_act_delete`删除它。

  如果有错误的话就会把错误复制给`r`的`error`属性。可以用`mpc_err_piint`,`mpc_err_delete`打印和删除错误。

  编译并运行上面的更新。尝试不同的输出看看系统是什么样的反应。正确的表现应该是像下面这样的。

  ```
Lispy Version 0.0.0.0.2
Press Ctrl+c to Exit

lispy> + 5 (* 2 2)
>
  regex
  operator|char:1:1 '+'
  expr|number|regex:1:3 '5'
  expr|>
    char:1:5 '('
    operator|char:1:6 '*'
    expr|number|regex:1:8 '2'
    expr|number|regex:1:10 '2'
    char:1:11 ')'
  regex
lispy> hello
<stdin>:1:1: error: expected whitespace, '+', '-', '*' or '/' at 'h'
lispy> / 1dog
<stdin>:1:4: error: expected one of '0123456789', whitespace, '-', one or more of one of '0123456789', '(' or end of input at 'd'
lispy>
  ```
#参考

parsing.c

```c
#include "mpc.h"

#ifdef _WIN32

static char buffer[2048];

char* readline(char* prompt) {
  fputs(prompt, stdout);
  fgets(buffer, 2048, stdin);
  char* cpy = malloc(strlen(buffer)+1);
  strcpy(cpy, buffer);
  cpy[strlen(cpy)-1] = '\0';
  return cpy;
}

void add_history(char* unused) {}

#else
#include <editline/readline.h>
#include <editline/history.h>
#endif

int main(int argc, char** argv) {
  
  /* Create Some Parsers */
  mpc_parser_t* Number   = mpc_new("number");
  mpc_parser_t* Operator = mpc_new("operator");
  mpc_parser_t* Expr     = mpc_new("expr");
  mpc_parser_t* Lispy    = mpc_new("lispy");
  
  /* Define them with the following Language */
  mpca_lang(MPCA_LANG_DEFAULT,
    "                                                     \
      number   : /-?[0-9]+/ ;                             \
      operator : '+' | '-' | '*' | '/' ;                  \
      expr     : <number> | '(' <operator> <expr>+ ')' ;  \
      lispy    : /^/ <operator> <expr>+ /$/ ;             \
    ",
    Number, Operator, Expr, Lispy);
  
  puts("Lispy Version 0.0.0.0.2");
  puts("Press Ctrl+c to Exit\n");
  
  while (1) {
  
    char* input = readline("lispy> ");
    add_history(input);
    
    /* Attempt to parse the user input */
    mpc_result_t r;
    if (mpc_parse("<stdin>", input, Lispy, &r)) {
      /* On success print and delete the AST */
      mpc_ast_print(r.output);
      mpc_ast_delete(r.output);
    } else {
      /* Otherwise print and delete the Error */
      mpc_err_print(r.error);
      mpc_err_delete(r.error);
    }
    
    free(input);
  }
  
  /* Undefine and delete our parsers */
  mpc_cleanup(4, Number, Operator, Expr, Lispy);
  
  return 0;
}

```

