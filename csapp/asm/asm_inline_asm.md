# C 与 内联汇编

# 简单的内联汇编

先看几个例子:

1. 来自 `https://github.com/Tencent/libco/blob/master/coctx.cpp`
```c
extern "C"
{
	extern void coctx_swap( coctx_t *,coctx_t* ) asm("coctx_swap");
};
```

2. 直接在 `__asm__("汇编代码");` 或 `asm("汇编")` 这样的格式写即可.

如果要使用 `-ansi` 或其他一些特定 `-std` 选项的话,可能不能使用 `asm` 
而需要使用 `__asm__`

# 扩展汇编
但是现实的示例中,大部分是扩展内联汇编.


扩展汇编中:
1. 可以通过汇编读和写C语言的变量.
2. 可以从汇编代码跳转到C标签中.
3. 扩展汇编语法中使用冒号`:`来在汇编代码模板之后分隔操作数.

扩展内联汇编格式1:

```
asm asm-qualifiers( AssemblerTemplate
  :OutputOperands
  [:InputOperands]
  [:Clobbers]
)
```

扩展内联汇编格式2:
```
asm asm-qualifiers( AssemblerTemplate
  :
  :InputOperands
  :Clobbers
  :GlobalLabels
)

```

在格式2中, `asm-qualifers` 包含 `goto`

# Qualifers

- `volatile`  asm 语句可能产生副作用,可以使用  `volatie` 修饰禁用某些优化.
- `inline` 
- `goto`

# Parameters

- `AssemblerTemplate` 汇编代码字符串,但是同时包含引用了输入,输出,和跳转参数的标识.

# 输出操作数
一条 `asm` 语句有0个或多个输出操作数.声明将被汇编代码修改的C变量.

操作数使用逗号分隔. 每一个操作数使用如下语法声明.
`[[asmSymbolicName]]constraint(cvariableName)`
`[[符号名]]约束(c语言变量名)`

注意:最外层的中括号表示里面的语法结构是可选的.

- `asmSymbolicName`  可选,为操作数指定操作符号名. 在汇编中通过 `%[符号名]` 的格式进行引用.
   一般不指定名称,那么可以使用 `%N` 的方式引用. `N` 表示操作数以0开始的索引.
   也就是第一个操作数使用`%0`来引用.

- `constraint` 以操作数的放置的一个约束字符串.  
  - 输出操作数必须以 `=` (表示写一个已经存在的值)开头.或 `+`(表示同时读和写)
  - 前缀的后面一般常用的约束包含 `r` 表示寄存器,`m` 表示内存. 也可以同时使用多个约束比如 `=rm`. 编译器会自动根据上下文选择最合适的.


# 简单约束符

- `r` 任意能用寄存器
- `m` 任意内存
- `o` 内存操作数可用,但是要求内存地址是可偏移的.(即可适用于基址寻址模式)
- `V` 不可偏移的内存操作数.
- `i` 一个立即数整型常量,(也包含编译器可知的符号常量)
- `n` 一个立即数整型常量,编译期不确定的也可以.
- `E` 一个立即浮点数 (要求与机器格式一样)
- `F` 一个立即浮点数
- `s` 一个立即整型,其值不是整型也可以.
- `g` 通用寄存器,内存,立即整型操作数都可以.
- `X` 任意操作数
- `0`,`1`,`2`,...,`9` 匹配指定操作数类型的可以.
- `p` 操作数合理的内存地址可以.

# x86 家族 特定的操作数放置约束指示符.
- `R` 老一代寄存器-指从 i386时代,就出现的8个寄存器如 (a,b,c,d,si,di,bp,sp)
- `q` 任意可使用 `rl` 访问的寄存器,32-位指 `a,b,c,d`, 64位指任何整型寄存器.
- `Q` 任意可使用 `rh` 访问的寄存器,`a,b,c,d`
- `a` a寄存器
- `b` b寄存器
- `c` c寄存器
- `d` d寄存器
- `S` si寄存器
- `D` di寄存器
- `A` `a` 和 `d` 寄存器. 用于指令会将返回结果放在 `ax:dx` 寄存器对上的指令.
- `U` 本调用会影响的整型寄存器
- `f` 任意 80387 浮点(栈)寄存器
- `t`  80387 浮点栈顶寄存器 (`%st(0)`)
- `u`  80387 浮点栈次顶寄存器 (`%st(1)`)
- `y` 任意 MMX 寄存器
- `x` 任意 SSE 寄存器
- `v` 任意 EVEX 编码的 SSE 寄存器 (`%xmm0 - %xmm31`)
- `Yz` 第一个 `SSE` 寄存器 (`%xmm0`)
- `I` 0到31的整型常量,用于 32位的位移.
- `J` 0到63的整型常量,用于 64位的位移.
- `K` 有符号的8位整型常量
- `L` 0xFF 或 `0xFFFF` 用于 `andsi` 作为以零扩展的移动.
- `M` 0,1,2或3 (用于 lea 指令的位移)
- `N` 无符号8位整型(用于 `in` `out` 指令)
- `G` 标准的 80387 浮点常量
- `C` SSE 常量0操作数
- `e` 32位有符号整型常量,或适用此范围的符号引用(适用以符号扩展的立即操作数)
- `We` 32位无符号,适用有符号扩展
- `Wz`  32位有符号,适用零扩展
- `Wd` 同上128位,要求低和高64位满足 `e` 的条件
- `Z` 同上32位 (用于以零扩展的立即操作数)
- `Tv` VSIB 地址操作数
- `Ts` 不带段的地址操作数

