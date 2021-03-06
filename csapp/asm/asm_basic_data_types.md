# 数据格式
在 Intel 术语体系中,他们用 word(16位,字)表示一个16位数据类型. 因为 32位便是 double word(双字),那么64位就是 quad word(四字)

下表是 C 语言数据类型在 x86-64 中的说明.(64位机器 指针长64位(8字节))

| C声明  | Intel数据类型     | 汇编代码后缀 | 大小(字节) |
| ------ | ----------------- | ------------ | ---------- |
| char   | 字节              | b            | 1          |
| short  | 字(word)          | w            | 2          |
| int    | 双字(double word) | l            | 4          |
| long   | 四字(quad word)   | q            | 8          |
| char * | 四字(quad word)   | q            | 8          |
| float  | 单精度            | s            | 4          |
| double | 双精度            | l            | 8          |

## 通用寄存器
| 64位 | 低32位 | 低16位 | 低8位 | 备注         |
| ---- | ------ | ------ | ----- | ------------ |
| %rax | %eax   | %ax    | %al   | 返回值       |
| %rbx | %ebx   | %bx    | %bl   | 被调用者保存 |
| %rcx | %ecx   | %cx    | %cl   | 第4个参数    |
| %rdx | %edx   | %dx    | %dl   | 第3个参数    |
| %rsi | %esi   | %si    | %sil  | 第2个参数    |
| %rdi | %edi   | %di    | %dil  | 第1个参数    |
| %rbp | %ebp   | %bp    | %bpl  | 被调用者保存 |
| %rsp | %esp   | %sp    | %spl  | 栈指针       |
| %r8  | %r8d   | %r8w   | %r8b  | 第5个参数    |
| %r9  | %r9d   | %r9w   | %r9b  | 第6个参数    |
| %r10 | %r10d  | %r10w  | %r10b | 调用者保存   |
| %r11 | %r11d  | %r11w  | %r11b | 调用者保存   |
| %r12 | %r12d  | %r12w  | %r12b | 被调用者保存 |
| %r13 | %r13d  | %r13w  | %r13b | 被调用者保存 |
| %r14 | %r14d  | %r14w  | %r14b | 被调用者保存 |
| %r15 | %r15d  | %r15w  | %r15b | 被调用者保存 |


复制生成值到寄存器,但是小于8字节的一些规则:
1. 生成1字节和2字节的数字的指令会保持剩下的字节不变.
2. 生成4字节的指令会把高4位字节设置为0.(为了兼容 IA32)

参数使用寄存器的顺序:`%rdi`,`%rsi`,`%rdx` ,`%rcx`,`%r8`,`%r9`