LoomTable# 8086 指令集及用法

本文档按功能分类列出标准 8086 指令集，每条指令包含功能简述、使用标准及代码范例。
对于有多种典型用法的指令，以 **用法 1**、**用法 2** 分列说明。

---

## 一、数据传送指令

### MOV — 传送

> 将源操作数复制到目标操作数。

- **语法**: `MOV dst, src`
- **限制**: 两操作数不可同时为内存；不可将立即数直接送段寄存器（须经通用寄存器中转）；CS 不可作目标。
- **影响标志**: 无

```asm
; 用法 1: 寄存器间传送
MOV     AX, BX
MOV     BL, AL

; 用法 2: 立即数送寄存器
MOV     CX, 100H
MOV     AL, '$'

; 用法 3: 内存 ↔ 寄存器
MOV     AX, [SI]        ; 内存 → 寄存器
MOV     [DI], AX        ; 寄存器 → 内存
MOV     AX, ARRAY[SI]   ; 基址+变址寻址

; 用法 4: 段寄存器初始化 (须中转)
MOV     AX, DATA
MOV     DS, AX
MOV     AX, STACK
MOV     SS, AX
```

### PUSH — 压栈

> 将 16 位操作数压入堆栈，SP 减 2。

- **语法**: `PUSH src`
- **限制**: 操作数必须为 16 位（寄存器、内存或段寄存器）；8086 不可压入立即数。
- **影响标志**: 无

```asm
; 用法 1: 保存寄存器
PUSH    AX
PUSH    CX
PUSH    SI

; 用法 2: 保存段寄存器 / 标志
PUSH    DS
PUSHF

; 用法 3: 远调用前压入参数 (段+偏移)
PUSH    CX              ; 压入目标段址
PUSH    BX              ; 压入目标偏移
RETF                    ; 远跳转 = 弹出 IP + CS

; 用法 4: 除十取余时暂存余数
XOR     DX, DX
DIV     BX
PUSH    DX              ; 将余数压栈，稍后逆序弹出输出
INC     CX
```

### POP — 出栈

> 从堆栈顶弹出 16 位数到目标操作数，SP 加 2。

- **语法**: `POP dst`
- **限制**: CS 不可作目标。
- **影响标志**: 无

```asm
; 用法 1: 恢复寄存器
POP     CX
POP     BX
POP     AX

; 用法 2: 恢复段寄存器
POP     DS
POP     ES

; 用法 3: 逆序输出除余结果
POP     DX              ; 后入先出，实现逆序
ADD     DL, '0'
MOV     AH, 02H
INT     21H
```

### XCHG — 交换

> 交换源与目标操作数的内容。

- **语法**: `XCHG dst, src`
- **限制**: 至少有一个操作数为寄存器；不可同时为内存。
- **影响标志**: 无

```asm
; 用法 1: 两寄存器交换
XCHG    AX, BX
XCHG    AH, AL

; 用法 2: 寄存器与内存交换
XCHG    AL, [SI]
```

### XLAT — 换码 / 查表

> 以 AL 为偏移量，从 BX 指向的表中取一字节替换 AL。

- **语法**: `XLAT` (操作数隐含为 `[BX+AL]`)
- **限制**: 使用前须将表基址装入 BX，索引装入 AL。
- **影响标志**: 无

```asm
; 用法: ASCII ↔ 数值转换
LEA     BX, HEX_TABLE
MOV     AL, 10
XLAT                    ; AL ← HEX_TABLE[10] = 'A'
```

### LEA — 装入有效地址

> 将源操作数的偏移地址（而非内容）装入目标寄存器。

- **语法**: `LEA reg16, mem`
- **限制**: 目标必须为 16 位通用寄存器。
- **影响标志**: 无

```asm
; 用法 1: 获取变量偏移地址
LEA     SI, BUF         ; SI ← BUF 的偏移地址
LEA     DX, PROMPT      ; DX ← PROMPT 偏移，用于 INT 21H/09H

; 用法 2: 加载栈顶指针
LEA     SP, TOP
```

### LDS / LES — 装入地址指针

> 从内存读取 32 位远指针：低 16 位送指定寄存器，高 16 位送 DS（LDS）或 ES（LES）。

- **语法**: `LDS reg16, mem32` / `LES reg16, mem32`
- **影响标志**: 无

```asm
; 用法: 加载远指针
LDS     SI, DWORD PTR [BX]      ; DS:SI ← [BX] 处的远指针
LES     DI, DWORD PTR [BP+2]    ; ES:DI ← [BP+2] 处的远指针
```

### LAHF / SAHF — 标志传送

> LAHF: 将标志寄存器低 8 位装入 AH。SAHF: 将 AH 存入标志寄存器低 8 位。

- **语法**: `LAHF` / `SAHF`
- **影响标志**: LAHF 无；SAHF 影响 SF, ZF, AF, PF, CF

```asm
LAHF                    ; AH ← flags 低 8 位 (SF:ZF:xx:AF:xx:PF:xx:CF)
SAHF                    ; flags 低 8 位 ← AH
```

### PUSHF / POPF — 标志寄存器压栈/出栈

- **语法**: `PUSHF` / `POPF`
- **影响标志**: PUSHF 无；POPF 影响全部标志

```asm
PUSHF                   ; 保存全部标志
CALL    SUBPROC
POPF                    ; 恢复全部标志
```

---

## 二、算术运算指令

### ADD — 加法

> 源操作数加到目标操作数，结果存目标。

- **语法**: `ADD dst, src`
- **影响标志**: OF, SF, ZF, AF, PF, CF

```asm
; 用法 1: 寄存器加法
ADD     AX, BX          ; AX ← AX + BX
ADD     AX, 100         ; AX ← AX + 100

; 用法 2: 内存操作数加法
ADD     [SI], AL        ; [SI] ← [SI] + AL
ADD     WORD PTR [DI], 2

; 用法 3: 多精度运算 (32 位符号扩展加法)
ADD     AX, BX          ; 加低 16 位
ADC     DX, BP          ; 加高 16 位，带进位
```

### ADC — 带进位加法

> dst ← dst + src + CF，用于多精度运算。

- **语法**: `ADC dst, src`
- **影响标志**: OF, SF, ZF, AF, PF, CF

```asm
; 用法 1: 32 位加法高位
ADD     AX, BX          ; 低 16 位相加
ADC     DX, 0           ; 高 16 位加进位

; 用法 2: 32 位有符号累加
ADD     AX, BX
ADC     DX, BP          ; BP 为 BX 的符号扩展
```

### INC — 加 1

> 操作数加 1，不影响 CF。

- **语法**: `INC dst`
- **影响标志**: OF, SF, ZF, AF, PF（不影响 CF）

```asm
; 用法 1: 循环计数器
INC     CX              ; CX++

; 用法 2: 指针自增
INC     SI              ; SI 指向下一字节
INC     DI

; 用法 3: 内存单元自增
INC     WORD PTR [BX]
INC     BYTE PTR POS_COUNT
```

### SUB — 减法

> 目标操作数减去源操作数，结果存目标。

- **语法**: `SUB dst, src`
- **影响标志**: OF, SF, ZF, AF, PF, CF

```asm
; 用法 1: 寄存器减
SUB     AX, 10          ; AX ← AX - 10

; 用法 2: ASCII 码转换为数值
SUB     AL, '0'         ; '5' → 5
SUB     BL, '0'

; 用法 3: 字母转大写
SUB     AL, 20H         ; 'a' → 'A'
```

### SBB — 带借位减法

> dst ← dst - src - CF。

- **语法**: `SBB dst, src`
- **影响标志**: OF, SF, ZF, AF, PF, CF

```asm
; 用法: 32 位减法高位
SUB     AX, BX          ; 低 16 位相减
SBB     DX, 0           ; 高 16 位减借位
```

### DEC — 减 1

> 操作数减 1，不影响 CF。

- **语法**: `DEC dst`
- **影响标志**: OF, SF, ZF, AF, PF（不影响 CF）

```asm
; 用法 1: 循环计数器
DEC     CX              ; 等价于 LOOP 的 CX-1 部分

; 用法 2: 指针回退
DEC     SI              ; SI 指向上一字节 (倒序输出时)
```

### MUL — 无符号乘法

> AL (8 位) 或 AX (16 位) 乘以源操作数，结果存入 AX (8 位) 或 DX:AX (16 位)。

- **语法**: `MUL src`
- **限制**: 源不可为立即数。若结果高位非零，CF=OF=1。
- **影响标志**: CF, OF（SF, ZF, AF 无定义）

```asm
; 用法 1: 8 位乘法
MOV     AL, 25
MOV     BL, 4
MUL     BL              ; AX ← AL × BL = 100

; 用法 2: 16 位乘法
MOV     AX, 1000
MOV     BX, 3000
MUL     BX              ; DX:AX ← AX × BX

; 用法 3: 32 位 × 16 位多精度乘法 (8086)
MOV     AX, LOW32       ; 32 位数的低 16 位
MOV     DX, HIGH32      ; 32 位数的高 16 位
MOV     BX, MULT16
PUSH    DX
MUL     BX              ; 低 16 位 × BX → DX:AX
MOV     RESULT, AX
MOV     RESULT+2, DX
POP     AX              ; 恢复高 16 位到 AX
MUL     BX              ; 高 16 位 × BX → DX:AX
ADD     AX, RESULT+2    ; 中间部分相加
ADC     DX, 0
MOV     RESULT+4, AX
MOV     RESULT+6, DX

; 用法 4: 十进制转换中的 ×10
MOV     AX, 10
MUL     BX              ; BX × 10 → DX:AX
```

### IMUL — 有符号乘法

> 与 MUL 类似，但操作数按有符号数处理。

- **语法**: `IMUL src`
- **限制**: 同 MUL。若结果高位是低位符号扩展，CF=OF=0；否则 CF=OF=1。
- **影响标志**: CF, OF

```asm
; 用法: 有符号乘法
MOV     AL, -5          ; FBH
MOV     BL, 3
IMUL    BL              ; AX ← -15 (FFF1H)
```

### DIV — 无符号除法

> AX (8 位除) 或 DX:AX (16 位除) 除以源操作数。商存入 AL (8 位) 或 AX (16 位)；余数存入 AH (8 位) 或 DX (16 位)。

- **语法**: `DIV src`
- **限制**: 若商超出寄存器范围，触发 0 号中断（除法溢出）。16 位除前须将 DX 清 0。
- **影响标志**: 无定义（SF, ZF, AF, PF, CF, OF 均不确定）

```asm
; 用法 1: 8 位除法
MOV     AL, 100
MOV     AH, 0           ; 被除数扩展到 AX
MOV     BL, 10
DIV     BL              ; AL = 10 (商), AH = 0 (余)

; 用法 2: 16 位无符号除法
XOR     DX, DX          ; 被除数高位置 0
MOV     AX, 65535
MOV     BX, 1000
DIV     BX              ; AX = 65 (商), DX = 535 (余)

; 用法 3: 除十取余 (数值转十进制)
XOR     DX, DX
MOV     BX, 10
DIV     BX              ; AX = 商, DX = 个位
PUSH    DX              ; 压栈余数
INC     CX              ; 计数
CMP     AX, 0
JNE     DIV_LOOP
```

### IDIV — 有符号除法

> 与 DIV 类似，但操作数按有符号数处理。余数符号与被除数相同。

- **语法**: `IDIV src`
- **限制**: 16 位除法前须用 CWD 将 AX 符号扩展到 DX:AX。
- **影响标志**: 无定义

```asm
; 用法: 有符号除法 (求平均值等)
MOV     AX, -100
CWD                     ; DX:AX ← 符号扩展的 -100
MOV     BX, 7
IDIV    BX              ; AX = -14 (商), DX = -2 (余)

MOV     CX, 100H
IDIV    CX              ; 32 位有符号和 ÷ 256 → AX=平均值
```

### CWD — 字扩展为双字

> 将 AX 的符号扩展到 DX（AX ≥ 0 则 DX=0，AX < 0 则 DX=FFFFH）。IDIV 16 位除法的前置指令。

- **语法**: `CWD`
- **影响标志**: 无

```asm
; 用法: IDIV 前的必做步骤
MOV     AX, -50
CWD                     ; DX:AX ← FFFF:FFCEH
IDIV    BX
```

### CBW — 字节扩展为字

> 将 AL 的符号扩展到 AH。IDIV 8 位除法的前置指令。

- **语法**: `CBW`
- **影响标志**: 无

```asm
; 用法: 8 位 IDIV 前
MOV     AL, -5          ; FBH
CBW                     ; AX ← FFFBH
IDIV    BL
```

### NEG — 求补 / 取负

> 对操作数按位取反后加 1（即 0 - 操作数）。

- **语法**: `NEG dst`
- **影响标志**: OF, SF, ZF, AF, PF, CF（结果为 0 时 CF=0，否则 CF=1）

```asm
; 用法 1: 有符号数取相反数
NEG     AX              ; AX ← -AX

; 用法 2: 取绝对值 (配合条件跳转)
CMP     AX, 0
JGE     ALREADY_POS
NEG     AX              ; AX ← |AX|
ALREADY_POS:

; 用法 3: 显示负数时取补
PUSH    AX
MOV     DL, '-'         ; 先输出负号
MOV     AH, 02H
INT     21H
POP     AX
NEG     AX              ; 转为正数再正常输出
```

### CMP — 比较

> 目标减源，仅影响标志位，不回写结果。常后跟条件跳转指令。

- **语法**: `CMP dst, src`
- **影响标志**: OF, SF, ZF, AF, PF, CF

```asm
; 用法 1: 与立即数比较 (字符范围判断)
CMP     AL, '0'
JB      NOT_DIGIT       ; AL < '0'
CMP     AL, '9'
JA      NOT_DIGIT       ; AL > '9'

; 用法 2: 寄存器间比较
CMP     AX, BX
JE      EQUAL           ; AX = BX
JG      GREATER         ; AX > BX (有符号)

; 用法 3: 与 0 比较 (正负判断)
CMP     AX, 0
JGE     POSITIVE        ; AX ≥ 0 (有符号)
JL      NEGATIVE        ; AX < 0

; 用法 4: 字符串结束判断
CMP     AL, '$'         ; 是否到达 '$' 结束符
JE      DONE
CMP     AL, 0DH         ; 是否到达回车
JE      INPUT_END
```

---

## 三、逻辑运算指令

### AND — 逻辑与

> dst ← dst AND src。常用于屏蔽特定位（清 0）。

- **语法**: `AND dst, src`
- **影响标志**: OF=0, CF=0, SF, ZF, PF

```asm
; 用法 1: 掩码保留低 nibble/bit
AND     AL, 0FH         ; 保留低 4 位，高 4 位清 0
AND     BL, 07H         ; 保留低 3 位 (00000111B)
AND     DL, 01H         ; 保留最低位

; 用法 2: 清除特定位
AND     CL, 0DFH        ; 清 CL 第 5 位 (11011111B)

; 用法 3: ASCII 转数值
AND     AL, 0FH         ; '5' (35H) → 5
AND     AX, 000FH       ; 同时清 AH + 提取 AL 低 4 位
```

### OR — 逻辑或

> dst ← dst OR src。常用于置位特定位。

- **语法**: `OR dst, src`
- **影响标志**: OF=0, CF=0, SF, ZF, PF

```asm
; 用法 1: 设置特定位
OR      CL, 20H         ; 置 CL 第 5 位 (00100000B)

; 用法 2: 数值转 ASCII
OR      BL, 30H         ; 数字 0-9 转为 ASCII '0'-'9'

; 用法 3: 合并 nibble 到寄存器
SHL     BX, 4
OR      BL, AL          ; 将 AL 的低 4 位并入 BX
```

### XOR — 异或

> dst ← dst XOR src。常用于寄存器清 0（比 MOV 更高效）。

- **语法**: `XOR dst, src`
- **影响标志**: OF=0, CF=0, SF, ZF, PF

```asm
; 用法 1: 寄存器清零 (比 MOV reg, 0 更紧凑)
XOR     AX, AX          ; AX ← 0
XOR     CX, CX          ; CX ← 0
XOR     BX, BX          ; BX ← 0

; 用法 2: 除前清 DX
XOR     DX, DX          ; 16 位 DIV 前置清高位
```

### NOT — 逻辑非

> dst ← NOT dst（按位取反）。

- **语法**: `NOT dst`
- **影响标志**: 无

```asm
; 用法: 位翻转
NOT     AL              ; AL ← 按位取反
```

### TEST — 测试 / 位测试

> 两操作数按位与，仅影响标志，不回写。常用于测试特定位是否为 0。

- **语法**: `TEST dst, src`
- **影响标志**: OF=0, CF=0, SF, ZF, PF

```asm
; 用法 1: 测试奇偶性
TEST    AL, 1           ; 测试 AL 最低位
JNZ     ODD             ; 非零 → 奇数

TEST    AX, 1
JZ      EVEN            ; 零 → 偶数

; 用法 2: 测试符号位
TEST    AX, 8000H       ; 测试最高位 (符号位)
JNZ     NEGATIVE        ; 最高位为 1 → 负数
```

---

## 四、移位与循环移位指令

> 移位次数可为 1 或 CL。8086 中仅支持 `SHL dst, 1` 或 `SHL dst, CL`。

### SHL / SAL — 逻辑左移 / 算术左移

> 将操作数左移 n 位，低位补 0，高位移入 CF。SHL 与 SAL 完全等价。

- **语法**: `SHL dst, 1` 或 `SHL dst, CL`
- **影响标志**: CF, OF, SF, ZF, PF

```asm
; 用法 1: 快速乘 2^n
SHL     BX, 1           ; BX ← BX × 2
MOV     CL, 4
SHL     AX, CL          ; AX ← AX × 16

; 用法 2: 按位移出逐位判断 (二进制输出)
MOV     CX, 16
SHL     BX, 1           ; 最高位 → CF
MOV     DL, '0'
ADC     DL, 0           ; DL = '0' + CF
INT     21H
LOOP    OUTPUT_LOOP

; 用法 3: 左移腾出空间 (十六进制数输入)
MOV     CL, 4
SHL     BX, CL          ; BX 左移 4 位腾出低位
OR      BL, AL          ; 并入新输入的数字

; 用法 4: 索引 ×4 (跳转表)
SHL     AX, 1
SHL     AX, 1           ; AX ← index × 4
```

### SHR — 逻辑右移

> 将操作数右移 n 位，高位补 0，低位移入 CF。

- **语法**: `SHR dst, 1` 或 `SHR dst, CL`
- **影响标志**: CF, OF, SF, ZF, PF

```asm
; 用法 1: 无符号快速除 2^n
SHR     BX, 1           ; BX ← BX / 2 (无符号)

; 用法 2: 处理 nibble 后移出 (十六进制/BCD 拆分)
MOV     CL, 4
SHR     AL, CL          ; AL 高 4 位 → 低 4 位

; 用法 3: 字计数转字节计数
SHR     CX, 1           ; CX ← CX / 2
```

### SAR — 算术右移

> 将操作数右移 n 位，高位用原符号位填充（保留符号），低位移入 CF。

- **语法**: `SAR dst, 1` 或 `SAR dst, CL`
- **影响标志**: CF, OF, SF, ZF, PF

```asm
; 用法 1: 有符号数除 2^n
SAR     AX, 1           ; AX ← AX / 2 (有符号)

; 用法 2: 16 位符号扩展到 32 位
MOV     BP, BX
SAR     BP, 15          ; BP ← BX 的符号扩展 (全 0 或全 1)
; 此时 BP:BX 为 32 位有符号数
```

### ROL — 循环左移

> 将操作数左移 n 位，移出的高位回填到低位，同时复制到 CF。

- **语法**: `ROL dst, 1` 或 `ROL dst, CL`
- **影响标志**: CF, OF

```asm
; 用法 1: 不丢失数据的 nibble 轮转 (十六进制输出)
MOV     CX, 4
PRINT_HEX:
        ROL     BX, 4           ; 高 4 位轮转到低 4 位
        MOV     DL, BL
        AND     DL, 0FH
        ADD     DL, '0'         ; 输出一位十六进制
        ...

; 用法 2: 八进制最高位移到最低位
ROL     BX, 1           ; 最高位 → 最低位
MOV     DL, BL
AND     DL, 01H
ADD     DL, '0'         ; 输出第一位二进制 (八进制最高位)
```

### ROR — 循环右移

> 将操作数右移 n 位，移出的低位回填到高位，同时复制到 CF。

- **语法**: `ROR dst, 1` 或 `ROR dst, CL`
- **影响标志**: CF, OF

```asm
; 用法 1: 交换 AH 与 AL
ROR     AX, 8

; 用法 2: nibble 右循环
ROR     BL, 4           ; 交换 BL 的高/低 4 位
```

### RCL — 带进位循环左移

> 左移 n 位，CF 参与循环（CF → 低位，高位 → CF）。

- **语法**: `RCL dst, 1` 或 `RCL dst, CL`
- **影响标志**: CF, OF

```asm
; 用法: 33 位移位 (配合 CF 做 17 位左移)
RCL     AX, 1
```

### RCR — 带进位循环右移

> 右移 n 位，CF 参与循环（CF → 高位，低位 → CF）。

- **语法**: `RCR dst, 1` 或 `RCR dst, CL`
- **影响标志**: CF, OF

```asm
; 用法: 33 位移位
RCR     AX, 1
```

---

## 五、串操作指令

> 源串地址默认 DS:SI，目标串地址默认 ES:DI。方向标志 DF=0 时地址递增（CLD），DF=1 时递减（STD）。

### MOVSB / MOVSW — 串传送

> 将 DS:SI 指向的字节/字传送到 ES:DI 指向的单元，并按 DF 方向调整 SI、DI。

- **语法**: `MOVSB` (字节) / `MOVSW` (字)
- **影响标志**: 无

```asm
; 用法 1: 重复前缀批量复制
CLD                     ; DF=0，地址递增
LEA     SI, SRC
LEA     DI, DST
MOV     CX, 100
REP     MOVSW           ; 复制 100 个字 (200 字节)

; 用法 2: 字节+字混合复制 (高效处理奇数字节)
MOV     CX, LEN
SHR     CX, 1           ; 字数量
REP     MOVSW           ; 先按字复制
ADC     CX, 0           ; 剩余 1 字节?
REP     MOVSB           ; 再复制剩余字节
```

### STOSB / STOSW — 串存储

> 将 AL (字节) 或 AX (字) 存入 ES:DI 指向的单元，并按 DF 方向调整 DI。

- **语法**: `STOSB` / `STOSW`
- **影响标志**: 无

```asm
; 用法: 批量填充
CLD
MOV     AX, 0
MOV     CX, 200
SHR     CX, 1
REP     STOSW           ; 将 200 字节填 0
```

### LODSB / LODSW — 串读取

> 从 DS:SI 指向的单元读取字节/字到 AL/AX，并按 DF 方向调整 SI。

- **语法**: `LODSB` / `LODSW`
- **影响标志**: 无

```asm
; 用法: 逐元素加载
CLD
LEA     SI, BUF
LODSB                   ; AL ← [SI], SI++
```

### CMPSB / CMPSW — 串比较

> DS:SI 与 ES:DI 指向的内容相减，仅影响标志位，同时调整 SI、DI。常配合 REPE/REPNE。

- **语法**: `CMPSB` / `CMPSW`
- **影响标志**: OF, SF, ZF, AF, PF, CF

```asm
; 用法: 字符串相等判断 (配合 REPE)
CLD
LEA     SI, STR1
LEA     DI, STR2
MOV     CX, LEN
REPE    CMPSB           ; 逐字节比较直到不等或 CX=0
JNE     MISMATCH        ; ZF=0 → 有不等字符
```

### SCASB / SCASW — 串扫描

> AL/AX 与 ES:DI 指向的内容相减，仅影响标志位，同时调整 DI。常配合 REPNE 搜索字符。

- **语法**: `SCASB` / `SCASW`
- **影响标志**: OF, SF, ZF, AF, PF, CF

```asm
; 用法: 搜索特定字符
CLD
MOV     AL, '$'         ; 搜索结束符
LEA     DI, STR
MOV     CX, 100
REPNE   SCASB           ; 逐字节搜索 $
JE      FOUND           ; ZF=1 表示找到
```

### REP / REPE / REPNE — 重复前缀

| 前缀 | 重复条件 |
|------|----------|
| `REP` | CX ≠ 0 时重复，CX 自减 |
| `REPE` (REPZ) | CX ≠ 0 且 ZF=1 时重复 |
| `REPNE` (REPNZ) | CX ≠ 0 且 ZF=0 时重复 |

```asm
; 用法 1: REP — 无条件串操作 (MOVS/STOS)
REP     MOVSB           ; 复制 CX 次

; 用法 2: REPE — 找不匹配 (CMPS)
REPE    CMPSB           ; 相等则继续比较
JNE     MISMATCH        ; 首次不匹配即跳出

; 用法 3: REPNE — 搜索匹配 (SCAS)
REPNE   SCASB           ; 不等则继续搜索
JE      FOUND           ; 找到目标
```

---

## 六、控制转移指令

### 无条件跳转

#### JMP — 无条件跳转

> 短跳转 (SHORT): ±127 字节内；近跳转 (NEAR): 段内 ±32K；远跳转 (FAR): 跨段。

- **语法**: `JMP label` (近) / `JMP FAR PTR label` (远) / `JMP reg16` (间接)
- **影响标志**: 无

```asm
; 用法 1: 段内短/近跳转
JMP     NEXT
JMP     INPUT_LOOP
JMP     EXIT

; 用法 2: 寄存器间接跳转
JMP     BX              ; IP ← BX
```

### 有条件跳转（按无符号比较）

> 所有条件跳转均为段内短跳转 (±127 字节)。**实用选型原则**：比较数值大小（地址、计数值、ASCII 码范围）→ 用 JA/JB；比较是否相等 → 用 JE/JNE。

| 指令 | 条件 | 等价含义 |
|------|------|----------|
| JE / JZ | ZF=1 | 相等 / 为 0 |
| JNE / JNZ | ZF=0 | 不等 / 非 0 |
| JA / JNBE | CF=0 且 ZF=0 | 高于 (无符号) |
| JAE / JNB / JNC | CF=0 | 高于或等于 / 无借位 |
| JB / JNAE / JC | CF=1 | 低于 (无符号) / 有借位 |
| JBE / JNA | CF=1 或 ZF=1 | 低于或等于 |

```asm
; 用法 1: 范围校验 (无符号)
CMP     AL, '0'
JB      NOT_DIGIT       ; AL < '0' (无符号)
CMP     AL, '9'
JA      NOT_DIGIT       ; AL > '9' (无符号)
; → '0' ≤ AL ≤ '9' 未跳转 → 数字字符

CMP     AL, 'a'
JB      EXIT            ; AL < 'a'
CMP     AL, 'z'
JA      EXIT            ; AL > 'z'
SUB     AL, 20H         ; 'a'-'z' → 转大写

; 用法 2: 相等比较
CMP     AX, BX
JE      EQUAL

; 用法 3: 无符号值与常数比较
CMP     BX, 10
JBE     SKIP            ; BX ≤ 10 跳过
```

### 有条件跳转（按有符号比较）

| 指令 | 条件 | 等价含义 |
|------|------|----------|
| JG / JNLE | ZF=0 且 SF=OF | 大于 (有符号) |
| JGE / JNL | SF=OF | 大于或等于 (有符号) |
| JL / JNGE | SF≠OF | 小于 (有符号) |
| JLE / JNG | ZF=1 或 SF≠OF | 小于或等于 (有符号) |

```asm
; 用法 1: 正负判断
CMP     AX, 0
JGE     POSITIVE        ; AX ≥ 0 (有符号)
JL      NEGATIVE        ; AX < 0 (有符号)

; 用法 2: 有符号值大小比较
CMP     DX, AX
JGE     NOT_EVEN        ; DX ≥ AX (有符号) 则跳过

; 用法 3: 绝对值比较后跳转 (注意: 用无符号)
CMP     DI, BX          ; 比较两个绝对值 (均为非负数)
JBE     SKIP            ; 用无符号跳转, 防 NEG(-32768) 误判
```

### 单个标志条件跳转

| 指令 | 条件 |
|------|------|
| JS | SF=1 (结果为负) |
| JNS | SF=0 (结果非负) |
| JO | OF=1 (有溢出) |
| JNO | OF=0 (无溢出) |
| JP / JPE | PF=1 (1 的个数为偶数) |
| JNP / JPO | PF=0 (1 的个数为奇数) |

```asm
; 用法: 符号位 / 溢出判断
TEST    AL, 80H
JS      NEGATIVE        ; 最高位为 1 → 负数
ADD     AX, BX
JO      OVERFLOW        ; 溢出则特殊处理
```

### LOOP — 循环控制

> CX ← CX - 1；若 CX ≠ 0，跳转到标号。

- **语法**: `LOOP label`
- **限制**: 目标在 ±127 字节内。
- **影响标志**: 无

```asm
; 用法 1: 定次循环
MOV     CX, 100
NEXT:
        ; ... 循环体 ...
        LOOP    NEXT    ; CX--, 若 CX≠0 则跳回

; 用法 2: CPU 自动管理计数器的循环
MOV     CX, 16
PRINT_BIT:
        SHL     BX, 1
        ; ... 逐位输出二进制 ...
        LOOP    PRINT_BIT
```

### LOOPE / LOOPZ — 相等则循环

> CX ← CX - 1；若 CX ≠ 0 且 ZF=1，跳转到标号。

```asm
; 用法: 搜索不相等元素
LOOPE   SCAN            ; 相等且未到头则继续
```

### LOOPNE / LOOPNZ — 不等则循环

> CX ← CX - 1；若 CX ≠ 0 且 ZF=0，跳转到标号。

```asm
; 用法: 搜索相等元素
LOOPNE  SEARCH          ; 不等且未到头则继续
```

### JCXZ — CX 为 0 跳转

> 若 CX=0 则跳转（不修改 CX）。

- **语法**: `JCXZ label`

```asm
; 用法 1: 空输入保护
JCXZ    EXIT            ; 若 CX=0 则直接退出循环

; 用法 2: 循环前安全检测
DEC     CX
JCXZ    SINGLE          ; 仅一个元素时直接跳到最后处理
```

### CALL — 过程调用

> 将返回地址压栈后跳转到目标过程。

- **语法**: `CALL proc` (近) / `CALL FAR PTR proc` (远)
- **验证**: 须确保 PUSH/POP 在子程序内成对出现。
- **影响标志**: 无

```asm
; 用法 1: 调用自定义子程序
CALL    PRINT_DEC       ; 段内近调用
CALL    INPUT_HEX       ; 输入十六进制子程序
CALL    OUTPUT_HEX      ; 输出十六进制子程序

; 用法 2: 递归调用
HTOA    PROC NEAR
        CALL    HTOA    ; 自身递归 (Hex to ASCII)
        ...
        RET
HTOA    ENDP
```

### RET / RETF — 过程返回

> RET: 近返回，从栈弹出 IP。RETF: 远返回，从栈依次弹出 IP 和 CS。
> RET n: 返回后 SP 额外加 n（用于清参）。

- **语法**: `RET` / `RET n` / `RETF`
- **影响标志**: 无

```asm
; 用法 1: 近过程返回
RET

; 用法 2: 远过程返回 (跨段)
RETF

; 用法 3: 带参数清栈返回
RET     4               ; 返回并弹出 4 字节参数
```

### INT — 软件中断

> 触发指定中断号。将 FLAGS、CS、IP 压栈后跳转到中断向量。

- **语法**: `INT n`
- **影响标志**: TF=0, IF=0

```asm
; 用法 1: DOS 功能调用
INT     21H             ; 最常用: AH=功能号, 其余寄存器传参

; 用法 2: 其他中断
INT     10H             ; 视频 BIOS 中断
INT     16H             ; 键盘 BIOS 中断
```

### IRET — 中断返回

> 从中断返回，依次弹出 IP、CS、FLAGS。

- **语法**: `IRET`
- **影响标志**: 全部（从栈恢复）

```asm
IRET                    ; 中断服务程序返回
```

---

## 七、处理器控制指令

### CLC / STC / CMC — 进位标志操作

> CLC: CF ← 0；STC: CF ← 1；CMC: CF ← ¬CF。

- **语法**: `CLC` / `STC` / `CMC`
- **影响标志**: CF

```asm
; 用法: 加法前清进位
CLC
ADC     AX, BX          ; 确保进位初始为 0 的带进位加法
```

### CLD / STD — 方向标志操作

> CLD: DF ← 0（地址递增）；STD: DF ← 1（地址递减）。

- **语法**: `CLD` / `STD`
- **影响标志**: DF

```asm
; 用法: 串操作前置方向设定
CLD                     ; 置为递增方向
REP     MOVSW           ; 按递增方向执行串传送
```

### CLI / STI — 中断标志操作

> CLI: IF ← 0（关中断）；STI: IF ← 1（开中断）。

- **语法**: `CLI` / `STI`
- **影响标志**: IF

```asm
; 用法: 临界区保护
CLI                     ; 关中断
; ... 临界操作 ...
STI                     ; 开中断
```

### NOP — 空操作

> 什么也不做。常用于延时、调试占位、对齐。

- **语法**: `NOP`
- **影响标志**: 无

```asm
NOP                     ; 占 1 字节，CPU 空转 1 周期
```

### HLT — 停机

> 暂停 CPU 直到中断或复位。

- **语法**: `HLT`
- **影响标志**: 无

```asm
HLT                     ; 等待中断
```

---

## 附录一：DOS INT 21H 功能速查表

以下涵盖本仓库所有 .asm 文件中实际使用的 INT 21H 功能号：字符 I/O (01H/02H/05H)、字符串 I/O (09H/0AH)、文件操作 (3CH/3DH/3EH/3FH/40H)、程序退出 (4CH)。

### AH = 01H — 带回显的字符输入

| 项目 | 说明 |
|------|------|
| 入口 | 无 |
| 出口 | AL = 输入的字符 (ASCII 码) |
| 行为 | 等待按键，字符回显在屏幕上；检测 Ctrl-C |
| 注意 | 若 AL=0 表示扩展键 (功能键/方向键)，需再读一次得扫描码 |

```asm
; 用法 1: 单字符输入
MOV     AH, 01H
INT     21H             ; AL = 输入字符

; 用法 2: 循环输入字符串直到回车
INPUT_LOOP:
        MOV     AH, 01H
        INT     21H
        CMP     AL, 0DH         ; 检测回车
        JE      INPUT_END
        MOV     [SI], AL        ; 存入内存
        INC     SI
        JMP     INPUT_LOOP

; 用法 3: 输入单个数字并验证
MOV     AH, 01H
INT     21H
CMP     AL, '0'
JB      EXIT
CMP     AL, '9'
JA      EXIT
SUB     AL, '0'         ; 转为数值
```

### AH = 02H — 字符输出

| 项目 | 说明 |
|------|------|
| 入口 | DL = 要显示的字符 (ASCII 码) |
| 出口 | 无 |
| 行为 | 在光标处显示 DL 中字符，光标后移；检测 Ctrl-C |

```asm
; 用法 1: 输出单个字符
MOV     DL, 'A'
MOV     AH, 02H
INT     21H

; 用法 2: 输出数字 (0-9 转 ASCII)
MOV     DL, CL
ADD     DL, '0'
MOV     AH, 02H
INT     21H

; 用法 3: 输出换行
MOV     DL, 0AH         ; 换行 LF
MOV     AH, 02H
INT     21H

; 用法 4: 输出响铃
MOV     DL, 07H         ; BELL 字符
MOV     AH, 02H
INT     21H

; 用法 5: 通过 ADC 逐位输出二进制
SHL     BX, 1
MOV     DL, '0'
ADC     DL, 0           ; DL = '0' 或 '1'
MOV     AH, 02H
INT     21H

; 用法 6: 循环输出字符串
OUTPUT_LOOP:
        MOV     DL, [SI]
        MOV     AH, 02H
        INT     21H
        INC     SI
        LOOP    OUTPUT_LOOP
```

### AH = 05H — 打印机字符输出

| 项目 | 说明 |
|------|------|
| 入口 | DL = 要打印的字符 (ASCII 码) |
| 出口 | 无 |
| 行为 | 将 DL 中字符发送到打印机 (PRN 设备)；若打印机忙则等待；检测 Ctrl-C |

```asm
; 用法 1: 打印单个字符
MOV     DL, 'A'
MOV     AH, 05H
INT     21H

; 用法 2: 打印字符串 (循环)
LEA     SI, MSG
PRINT_LOOP:
        MOV     DL, [SI]
        CMP     DL, '$'
        JE      DONE
        MOV     AH, 05H
        INT     21H
        INC     SI
        JMP     PRINT_LOOP
```

### AH = 09H — 字符串输出

| 项目 | 说明 |
|------|------|
| 入口 | DS:DX = 字符串首地址（字符串必须以 `$` 结尾） |
| 出口 | 无 |
| 行为 | 从 DS:DX 开始逐字符显示，直到遇到 `$`；`$` 本身不显示 |
| 注意 | `$` 是终止符，不是长度计数值；若字符串不含 `$` 会输出垃圾数据 |

```asm
; 用法 1: 定义时内嵌 '$'
DATA SEGMENT
    MSG DB 'Hello, World!$'

CODE SEGMENT
    LEA     DX, MSG
    MOV     AH, 09H
    INT     21H

; 用法 2: '$' 后附换行
    PROMPT DB 'SUN: $'             ; 前缀后 $

; 用法 3: 字符串比较后输出匹配结果
    MATCH_MSG    DB 'MATCH$'
    NO_MATCH_MSG DB 'NO MATCH$'
    ...
    LEA     DX, MATCH_MSG
    MOV     AH, 09H
    INT     21H
```

### AH = 0AH — 缓冲字符串输入

| 项目 | 说明 |
|------|------|
| 入口 | DS:DX = 缓冲区首地址 |
| 出口 | 缓冲区第 2 字节 = 实际输入字符数 (不含回车) |
| 行为 | 从键盘读取一行，直到回车。字符存入缓冲区从第 3 字节开始 |

**缓冲区格式**:

| 偏移 | 大小 | 含义 |
|------|------|------|
| [DX+0] | 1 字节 | 最大可输入字符数 (由调用者预设) |
| [DX+1] | 1 字节 | 实际输入字符数 (由 DOS 回填，不含 CR) |
| [DX+2] | N 字节 | 输入的字符串内容 |

```asm
; 用法 1: 标准缓冲输入
DATA SEGMENT
    BUF DB 100, 0, 100 DUP(?)    ; 最大 100, 实际长度 0, 缓冲区

CODE SEGMENT
    LEA     DX, BUF
    MOV     AH, 0AH
    INT     21H
    MOV     CL, BUF[1]            ; CL = 实际输入长度
    MOV     CH, 0
    LEA     SI, BUF[2]            ; SI 指向字符串内容

; 用法 2: 配合倒序输出
    MOV     CL, BUF[1]
    MOV     CH, 0
    LEA     SI, BUF[2]
    ADD     SI, CX                ; SI 指向末尾+1
PRINT_LOOP:
    DEC     SI
    MOV     DL, [SI]
    MOV     AH, 02H
    INT     21H
    LOOP    PRINT_LOOP
```

### AH = 4CH — 带返回码终止程序

| 项目 | 说明 |
|------|------|
| 入口 | AL = 返回码 (0=正常) |
| 出口 | 不返回 |
| 行为 | 关闭所有打开文件，释放内存，返回 DOS；可在批处理中用 ERRORLEVEL 检测返回码 |

```asm
; 用法 1: 正常退出 (标准 EXE 出口)
        MOV     AH, 4CH
        INT     21H

; 用法 2: 带错误码退出
        MOV     AL, 1
        MOV     AH, 4CH
        INT     21H
```

> **提示**: 本仓库所有 .asm 均使用 `MOV AH, 4CH` / `INT 21H` 作为 EXE 程序标准退出方式。对比 `.COM` 格式使用 `RET` 或 `INT 20H`。

### AH = 3CH — 创建或截断文件

| 项目 | 说明 |
|------|------|
| 入口 | DS:DX = 文件名 (ASCIZ 字符串，以 0 结尾)；CX = 文件属性 (0=普通) |
| 出口 | 成功 CF=0，AX = 文件句柄；失败 CF=1，AX = 错误码 |
| 行为 | 若文件不存在则创建；若已存在则将长度截为 0 |

```asm
; 用法 1: 创建普通文件
DATA SEGMENT
    FNAME DB 'd:\abc.txt', 0      ; ASCIZ 文件名

CODE SEGMENT
    LEA     DX, FNAME
    MOV     AH, 3CH
    MOV     CX, 0                   ; 普通属性
    INT     21H
    MOV     HANDLE, AX              ; 保存句柄供后续写入

; 用法 2: 覆盖已有文件
    MOV     CX, 0                   ; 3CH 会清空已有内容
    INT     21H                     ; 相当于 "重新创建"
```

### AH = 3DH — 打开文件

| 项目 | 说明 |
|------|------|
| 入口 | DS:DX = 文件名 (ASCIZ)；AL = 访问模式 (0=读, 1=写, 2=读写) |
| 出口 | 成功 CF=0，AX = 文件句柄；失败 CF=1，AX = 错误码 |

```asm
; 用法 1: 以只读方式打开
LEA     DX, FNAME
MOV     AH, 3DH
MOV     AL, 0                       ; 0 = 只读
INT     21H
JNC     OPEN_OK                     ; CF=0 成功
; ... 错误处理 ...

; 用法 2: 检查 CF 判断文件是否存在
MOV     AH, 3DH
MOV     AL, 0
INT     21H
JC      NOT_FOUND                   ; CF=1 表示打开失败
MOV     HANDLE, AX
```

### AH = 3EH — 关闭文件

| 项目 | 说明 |
|------|------|
| 入口 | BX = 文件句柄 |
| 出口 | 成功 CF=0；失败 CF=1，AX = 错误码 |
| 行为 | 刷新缓冲区，将句柄归还系统 |

```asm
; 用法: 关闭单个文件
MOV     BX, HANDLE
MOV     AH, 3EH
INT     21H

; 关闭多个文件 (逐一关闭)
MOV     BX, HAND1
MOV     AH, 3EH
INT     21H
MOV     BX, HAND2
MOV     AH, 3EH
INT     21H
```

### AH = 3FH — 从文件读取

| 项目 | 说明 |
|------|------|
| 入口 | BX = 文件句柄；CX = 要读取的字节数；DS:DX = 缓冲区 |
| 出口 | 成功 CF=0，AX = 实际读取的字节数；AX < CX 通常表示已到文件尾 (EOF)；AX=0 表示确已到文件尾 |
| 行为 | 从文件当前位置读取 CX 个字节到缓冲区，文件指针后移 |

```asm
; 用法 1: 读取固定字节数
MOV     BX, HANDLE
MOV     AH, 3FH
LEA     DX, BUF
MOV     CX, 10                      ; 读取 10 字节
INT     21H
MOV     CX, AX                      ; CX = 实际读到的字节数

; 用法 2: 分块循环读取整个文件
READ_LOOP:
        MOV     BX, HANDLE
        MOV     AH, 3FH
        LEA     DX, BUF
        MOV     CX, 256             ; 每次读 256 字节
        INT     21H
        CMP     AX, 0
        JBE     EOF                 ; AX=0 → 文件结束
        ; ... 处理 BUF 中的 AX 字节 ...
        JMP     READ_LOOP
```

### AH = 40H — 向文件写入

| 项目 | 说明 |
|------|------|
| 入口 | BX = 文件句柄；CX = 要写入的字节数；DS:DX = 数据缓冲区 |
| 出口 | 成功 CF=0，AX = 实际写入的字节数；若 AX < CX 可能磁盘已满 |
| 行为 | 将 CX 个字节从缓冲区写入文件当前位置，文件指针后移 |

```asm
; 用法 1: 写入固定数据
MOV     BX, HANDLE
MOV     AH, 40H
LEA     DX, DATA_BUF
MOV     CX, DATA_LEN
INT     21H

; 用法 2: 将键盘输入写入文件
LEA     DX, INBUF
MOV     AH, 0AH                     ; 先缓冲输入
INT     21H
MOV     BL, INBUF[1]                ; 取实际长度
XOR     BH, BH
MOV     CX, BX
MOV     BX, HANDLE
MOV     AH, 40H
LEA     DX, INBUF[2]                ; 跳过缓冲区头部
INT     21H

; 用法 3: 分块循环写入 (文件转换/拷贝)
WRITE_LOOP:
        ; ... 处理 BUF 中 CX 字节 ...
        MOV     BX, DST_HAND
        MOV     AH, 40H
        LEA     DX, BUF
        INT     21H
        ; ... 继续读取下一块 ...
```

---

## 附录二：标志位速查

| 标志 | 英文名 | 含义 | 常用于 |
|------|--------|------|--------|
| CF | Carry Flag | 进位/借位 | 多精度加减、移位溢出、条件跳转 (JB/JC) |
| PF | Parity Flag | 低 8 位中 1 的个数为偶数 | 通信校验 |
| AF | Auxiliary Carry | BCD 运算低 4 位向高 4 位进位 | DAA/DAS |
| ZF | Zero Flag | 运算结果为 0 | 相等比较 (JE)、循环结束判断 |
| SF | Sign Flag | 结果最高位 (符号) | 正负判断 (JS/JNS)、有符号比较 |
| TF | Trap Flag | 单步中断 | 调试 (DEBUG T 命令) |
| IF | Interrupt Flag | 允许可屏蔽中断 | CLI/STI |
| DF | Direction Flag | 串操作地址方向 | CLD (递增)/STD (递减) |
| OF | Overflow Flag | 有符号数溢出 | JO/JNO、有符号比较 (JL/JG) |
