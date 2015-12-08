#树

  现在可以读取输入，并且也有了内部结构，但我们还是不能计算它们。这节我们会添加一些代码来计算这些结构并且真正通过内部编码计算它们的值。

  这个内部结构救赎我们在前一节中打印出来的那些。被称之为抽象语法树，它代表了基于用户输入的程序的结构。这个树的叶子是数字和操作符，也就是真正需要被处理的数据。树叉是用来产生这些部分的规则，这些规则用来翻译和计算的信息。

  ![Abstract Christmas Tree A seasonal variation](./tree.png)

  在我们弄明白究竟我们打算怎样遍历之前，先来看看这些结构在内部是怎样定义的吧。浏览`mpc.h`文件，我们可以看到`mpc_ast_t`的定义，它是我们从解析器得来的一个数据结构。

```c
    typedef struct mpc_ast_t {
        char* tag;
        char* contents;
        mpc_state_t state;
        int children_num;
        struct mpc_ast_t** children;
    }mpc_ast_t;
```

  这个结构有很多我们可以访问的属性，让我们一个个的看下。

  第一个属性是`tag`。当我们打印树时，它是某个节点的前置信息。它是包含一个用来解析特点项的规则列表。比如`expr|number|regex`。

  这个`tag`属性对于我们查看用了哪些规则构建这个节点是很重要的。

  第二个属性是`contents`。它存储了这个节点的真正内容比如`*`,`(`，或者`5`等等。你会发现对于分支这个属性是空的，但对于叶子节点我们可以找到要用的操作符或者数字。

  下一个属性是`state`。这个部分包含了解析器现在的状态以及什么时候发现的这个节点，比如行号和列号。在我们的程序中是不会用到这部分的。

  最后我们看看帮助我们翻译树的俩个属性。`children_num`和`children`，第一个属性告诉我们一个节点有多少子节点，第二个是这些子节点的列表。

  `children`是一个`mpc_ast_t**`类型。这是一个双重指针，它并没有看起来那么吓人，我们在后面的章节中会详细的讲它的细节。现在你只需要知道它是子节点的列表就好了。

  我们可以用数组访问这个属性来访问节点。只需要像这样`children[0]`就可以访问了，注意C的计数是从`0`开始的。

  因为`mpc_ast_t*`是一个指向结构体的指针，所以访问它的属性时稍微有些不一样。得使用`->`而不是`.`。

  ```c
    mpc_ast_t* a = r.output;
    printf("Tag: %s\n",a->tag);
    printf("Contents: %s\n",a->contents);
    printf("Number of children: %i\n",a->children_num);
    mpc_ast_t* c0 = a->children[0];
    printf("First Child Tag: %s\n",c0->tag);
    printf("First Child Contents: %s\n",c0->contents);
    printf("First Child Children: %i\n",c0->children_num);
  ```

#递归

  树形结构有个特点，就是它们可以指向树。每个节点的子节点都是树，而这个树的子节点还是树。就像我们的语言和复写规则，这些结构包含和自己结构一样的子机构。

  这种重复子结构的模式可以不断重复。显然我们想要一个可以接受任何树的函数，我们就不能只搜寻一些节点就结束，我们需要让它在所有深度的树上都工作。

  幸运的是我们可以用递归来做到这点。

  递归函数的一个简单解释就是在某些条件下自己调用自己。

  一个函数自己调用自己听起来很诡异。但考虑到函数可以在有不同的输入递归调用函数并有结束条件，那么我们可以肯定的说递归函数是有用的。

  我们可以写一个可以递归遍历树上所有节点的函数。

  先从一个最简单的情形开始，也就是如果输入的树没有子节点。在这种情形下显然只有一个节点。现在看看更复杂的情形吧，如果树有一个或多个节点。这是结果是节点本身和子节点数量的值。

  但怎么定义所有子节点的数量呢？是的没错就是用递归。

  在C语言中可以像这样：

  ```c
  int number_of_nodes(mpc_ast_t* t){
    if (t->children_num == 0) { return 1; }
    if (t->children_num >= 1) {
        int total = 1;
        for (int i=0;i<t->children_num;i++){
            total = total + number_of_nodes(t->children[i]);
        }
        return total;
    }
  }
  ```

  因为递归函数需要奇怪的循环，所以它看起来有些怪。首先我们假设我们已经写好了可以正确工作的函数了，然后我们打算使用这个函数，写一个初始函数。

  像很多事情一样，递归函数基本遵循着相同的模式。首先定义一个基本情形。这个情形时递归结束条件，比如`t->children_num == 0`。接着是递归情形，比如`t->children_num >= 1`。

  递归函数需要慢慢体会，现在可以先停一下，确定你明白了再往下读，因为后面部分它的应用会很多。如果你不是很确定你是否明白了，你可以试一下这节的附加题。

#计算

  我们将会写一个递归函数来计算解析树。但开始前，我们先看看输入的树结构都有些需要注意。试着用上节的代码打印一下表达式，你都发现了什么？

  ```
lispy> * 10 (+ 1 51)
>
  regex
  operator|char:1:1 '*'
  expr|number|regex:1:3 '10'
  expr|>
    char:1:6 '('
    operator|char:1:7 '+'
    expr|number|regex:1:9 '1'
    expr|number|regex:1:11 '51'
    char:1:13 ')'
  regex
  ```

  首先一个节点如果被打上了`number`标签，那它永远都是number，没有子节点，而且可以直接转位整数。这是递归的基本情形。

  如果标签是`expr`，我们就得看看它的第二个子节点(因为它的第一个子节点永远都是`'('`)是什么操作符。然后我们需要把这个操作符作用到剩余的子节点，最后的节点总是`')'`，这是我们的递归条件

  检测节点的标签，或者从节点获得节点的数值时，我们需要充分利用`tag``contents`属性。它们都是字符串，因此我来讲一字符串的操作函数。

  `atoi`  把`char*`转成`long`

  `strcmp`  接受两个`char*`如果相等返回`0`

  `strstr`  接受两个`char*`返回第二个字符串在第一个字符串中的位置，如果第二个字符串不是第一个字符串的子串则返回0

  用`strcmp`检查有的是哪个操作符，用`strstr`检查标签是否包含某些子串。结合我们的计算和递归后函数应该像下面这样：

  ```c
long eval(mpc_ast_t* t) {
  
  /* If tagged as number return it directly. */ 
  if (strstr(t->tag, "number")) {
    return atoi(t->contents);
  }
  
  /* The operator is always second child. */
  char* op = t->children[1]->contents;
  
  /* We store the third child in `x` */
  long x = eval(t->children[2]);
  
  /* Iterate the remaining children and combining. */
  int i = 3;
  while (strstr(t->children[i]->tag, "expr")) {
    x = eval_op(x, op, eval(t->children[i]));
    i++;
  }
  
  return x;  
}
  ```
  
  当我们像下面这样定义`eval_op`函数时，它接受一个数字，一个操作符，以及另一个数字。它测试传来的参数是什么，并进行对应的运算。

  ```c
  long eval_op(long x, char* op, long y) {
  if (strcmp(op, "+") == 0) { return x + y; }
  if (strcmp(op, "-") == 0) { return x - y; }
  if (strcmp(op, "*") == 0) { return x * y; }
  if (strcmp(op, "/") == 0) { return x / y; }
  return 0;
  }   
  ```

#打印

  我们现在想要打印的不是树，而是计算的结果。先把树传给`eval`函数然后打印结果。

  还有要在计算后记得删除输出树

  ```c
long result = eval(r.output);
printf("%li\n", result);
mpc_ast_delete(r.output);

  ```

  如果一切顺利的话应该会使下面这样

  ```
Lispy Version 0.0.0.0.3
Press Ctrl+c to Exit

lispy> + 5 6
11
lispy> - (* 10 10) (+ 1 1 1)
97
  ```

#参考

evaluation.c

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

/* Use operator string to see which operation to perform */
long eval_op(long x, char* op, long y) {
  if (strcmp(op, "+") == 0) { return x + y; }
  if (strcmp(op, "-") == 0) { return x - y; }
  if (strcmp(op, "*") == 0) { return x * y; }
  if (strcmp(op, "/") == 0) { return x / y; }
  return 0;
}

long eval(mpc_ast_t* t) {
  
  /* If tagged as number return it directly. */ 
  if (strstr(t->tag, "number")) {
    return atoi(t->contents);
  }
  
  /* The operator is always second child. */
  char* op = t->children[1]->contents;
  
  /* We store the third child in `x` */
  long x = eval(t->children[2]);
  
  /* Iterate the remaining children and combining. */
  int i = 3;
  while (strstr(t->children[i]->tag, "expr")) {
    x = eval_op(x, op, eval(t->children[i]));
    i++;
  }
  
  return x;  
}

int main(int argc, char** argv) {
  
  mpc_parser_t* Number = mpc_new("number");
  mpc_parser_t* Operator = mpc_new("operator");
  mpc_parser_t* Expr = mpc_new("expr");
  mpc_parser_t* Lispy = mpc_new("lispy");
  
  mpca_lang(MPCA_LANG_DEFAULT,
    "                                                     \
      number   : /-?[0-9]+/ ;                             \
      operator : '+' | '-' | '*' | '/' ;                  \
      expr     : <number> | '(' <operator> <expr>+ ')' ;  \
      lispy    : /^/ <operator> <expr>+ /$/ ;             \
    ",
    Number, Operator, Expr, Lispy);
  
  puts("Lispy Version 0.0.0.0.3");
  puts("Press Ctrl+c to Exit\n");
  
  while (1) {
  
    char* input = readline("lispy> ");
    add_history(input);
    
    mpc_result_t r;
    if (mpc_parse("<stdin>", input, Lispy, &r)) {
      
      long result = eval(r.output);
      printf("%li\n", result);
      mpc_ast_delete(r.output);
      
    } else {    
      mpc_err_print(r.error);
      mpc_err_delete(r.error);
    }
    
    free(input);
    
  }
  
  mpc_cleanup(4, Number, Operator, Expr, Lispy);
  
  return 0;
}

```
