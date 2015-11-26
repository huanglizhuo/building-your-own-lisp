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
  
  用我们已有的知识足够编码上述的大部分规则，但 number 和 program 可能会有些麻烦。它们包含很多我们不知道怎样表达的结构。我们不知道怎么表达输入的开始，结束，可选的字符，或者字符的长度。

  这些也是可以表达的，但需要用正则表达式

