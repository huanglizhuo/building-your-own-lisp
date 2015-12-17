#崩溃

  你可能注意到前面章节的问题。试着把下面的代码输入到程序中，看看发生了什么。

  ```c
Lispy Version 0.0.0.0.3
Press Ctrl+c to Exit

lispy> / 10 0
  ```

  oh no 因为除零程序崩溃了！！！程序开发时崩溃还好，但最终程序最好用不崩溃，并可以告诉用户哪里出问题了。这时候我们的程序产可以预测语法错误但仍然没有在计算表达式时报告错误的函数。我们需要建立一些错误处理机制来函数式的处理错误。在C语言中这有些尴尬，但如果从正确的轨道开始，会对我们后面程序变的复杂时有很大的好处。

  C程序如果有任何错误操作系统就会把它踢出去。程序崩溃的原因有很多，并且方式也都不一样。

  但C程序运行中并没有什么魔法，如果你遇到一些艰难的bug不要放弃或者死死盯着它，这时你应该学习一下`gdb`和`valgrind`。这些是很强大的工具，在初始学习后，你会发现它们会节省你大量的时间。

  ![Walter White heisenberg][./walterwhite.png]

#lisp值

  在C语言中有很对处理错误的方式，但这里我推荐返回一个计算的可能值。这样就可以做到在 lispy ，一个表达式的结果要么是一个数字要么是一个错误。比如`+ 1 2` 返回一个数字，而`/ 10 0` 返回一个错误。

  为了做到这一点，我们需要一个数据结构可，要么是任何值要么是一个特定的值。最简单的方式就是用一个带有一个可以放任何值的属性并且可以呈现出来，一个一个`type`属性来给出哪些是由意义的属性。

  接下来我们打算叫它`lval`，代表 lisp 类型。

  ```c
    typedef struct{
        int type;
        int num;
        int err;
    }lval;
  ```

#枚举

  你可能注意到了`type` `err`字段的类型是 `int` 。

  我们之所以用`int`是因为我们将会给每个整数值赋予对应的意义。比如可以定义这样的规则如果`type`是`0`那么就是一个数字，或者如果`type`是`1`那么就是`error` 。

  但如果后面忘记这个对应的意义就会很麻烦，对此我们可以用命名常量来标识它。这样在比较的时候就可可以清楚的知道对应的值以及上下文了。

  在C语言中可以使用`enum`

  ```c

    enum{ LVAL_NUM, LVAL_ERR};
  ```

  每个`enum`会自动分配一个整数值。
  
  我们也可以定义一些错误类型，来表示除零错误溢出错误等。

  ```c
enum { LERR_DIV_ZERO, LERR_BAD_OP, LERR_BAD_NUM };
  ```

#lisp类型函数

  我们的`lval`类型基本准备好了但不像之前的`long`类型，我们的类型没有一个创建它的正确方法。为了做到这点，我们声明了两个函数用来把`lval`构造成要么是 error 要么是 number 类型.

  ```c
lval lavl_num(long x){
    lval v;
    v.type = LVAL_NUM;
    v.num = x;
    return v;    
}

lval lval_err(int x){
    lval v;
    v.type = LVAL_ERR;
    v.err = x;
    return v;
}
  ```
  
  这两个函数都是创建一个`lval`变量，然后根据接受的参数给它们赋值。

  因为`lval`函数可以是 error 或者 number 中的任意一个，所以我们不能简单的用`printf`来输出它了。
我们想要根据不同的类型决定不同的表现。在C中可以用`switch`语句做到这一点。它接受一个值作为输入并和其它已知的值做比较，称为case 。当比较的值相同时则执行对应的代码，直到碰到`break`语句。

  这样我们就可以打印任何`lval`的值了。

```c

/* Print an "lval" */
void lval_print(lval v) {
  switch (v.type) {
    /* In the case the type is a number print it */
    /* Then 'break' out of the switch. */
    case LVAL_NUM: printf("%li", v.num); break;

    /* In the case the type is an error */
    case LVAL_ERR:
      /* Check what type of error it is and print it */
      if (v.err == LERR_DIV_ZERO) {
        printf("Error: Division By Zero!");
      }
      if (v.err == LERR_BAD_OP)   {
        printf("Error: Invalid Operator!");
      }
      if (v.err == LERR_BAD_NUM)  {
        printf("Error: Invalid Number!");
      }
    break;
  }
}

/* Print an "lval" followed by a newline */
void lval_println(lval v) { lval_print(v); putchar('\n'); }
```
#计算错误

  现在我们知道怎样使用`lval`函数了，接下来需要改变我们的计算函数，让它接收`lval`类型而不是`long`

  在`eval_op`函数中，如果我们遇到 error 我们应该立即返回，只有在两个参数都是数字时才进行计算。在遇到除零时，我们应该返回错误而不是试着去计算。这会修复这章开头提过的崩溃。

  ```c
lval eval_op(lval x, char* op, lval y) {

  /* If either value is an error return it */
  if (x.type == LVAL_ERR) { return x; }
  if (y.type == LVAL_ERR) { return y; }

  /* Otherwise do maths on the number values */
  if (strcmp(op, "+") == 0) { return lval_num(x.num + y.num); }
  if (strcmp(op, "-") == 0) { return lval_num(x.num - y.num); }
  if (strcmp(op, "*") == 0) { return lval_num(x.num * y.num); }
  if (strcmp(op, "/") == 0) {
    /* If second operand is zero return error */
    return y.num == 0 
      ? lval_err(LERR_DIV_ZERO) 
      : lval_num(x.num / y.num);
  }

  return lval_err(LERR_BAD_OP);
}
  ```

  这里的`?`干了什么？

  你可能注意到,我们在判断除法指令的第二个参数是否是0时用了`?`符号，并跟着一个`:`。这是一个三元操作符，它允许你仅用一行代码进行条件测试。

  它工作过程就像`<confition> ? <then> : <else>`。也就是说，如果条件为真就返回`?`后面的否则返回`:`后面的。

  有的人不喜欢这个操作符，他们认为这会使代码不清晰。

  我们需要给`eval`函数也赋予类似的操作。因为我们的`eval_op`定义的很强健，所以我们只需要给数字转换函数添加错误条件。

  用`strtol`函数将字符串转为`long`。这允许我们检查一个特殊的变量`errno`来确定转换是正确的。这比我们之前用的`atoi`函数更强健。

  ```c
lval eval(mpc_ast_t* t) {
  
  if (strstr(t->tag, "number")) {
    /* Check if there is some error in conversion */
    errno = 0;
    long x = strtol(t->contents, NULL, 10);
    return errno != ERANGE ? lval_num(x) : lval_err(LERR_BAD_NUM);
  }
  
  char* op = t->children[1]->contents;  
  lval x = eval(t->children[2]);
  
  int i = 3;
  while (strstr(t->children[i]->tag, "expr")) {
    x = eval_op(x, op, eval(t->children[i]));
    i++;
  }
  
  return x;  
}
  ```

  最后的这一小步是改变我们新定义的`lval`类型的打印函数，使之可以正确打印。

  ```c
    lval result = eval(r.output);
    lval_println(result);
    mpc_ast_delete(r.output);
  ```

  好了做完了！现在试试我们的程序看它是否会在除0时崩溃

  ```
lispy> / 10 0
Error: Division By Zero!
lispy> / 10 2
5
  ```

##plumbing

  有些人会对现在的过程很不舒服。你可能觉得你只是按照指导做，但并不明白在这背后的原理。

  If this is the case I want to reassure you that you are doing well. If you don't understand the internals it's because I may not have explained everything in sufficient depth. This is okay.

  To be able to progress and get code to work under these conditions is a great skill in programming, and if you've made it this far it shows you have it.

  在编程中我们称之为plumbing。粗略来讲就是按照指示把很多库或者组件连接到一起，但并不明白它们内部是如何工作的。
