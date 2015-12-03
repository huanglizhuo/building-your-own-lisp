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

