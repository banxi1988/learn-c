# 函数,运行时栈,及参数传递

1. 不大于6个整型参数(包括指针)可以通过寄存器传递.
2. 超过部分通过调用栈传递(属于调用者的栈部分)
3. 简单的返回值通过 `rax` 传递.
  