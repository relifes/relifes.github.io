---
title: 汇编排序
toc: true
permalink: /posts/ASM-SORT/
date: 2022-03-05 16:41:59
categories: 汇编
tags: 汇编
---

## 题目描述

{% cq %}

**在键盘输入任意10个数**

1. **按从小到大排序后，在计算机屏幕上先输出来。要有结果提示（字符串显示）。**
2. **将10个数做累加，结果在计算机屏幕显示累加和。**

{% endcq %}

## 伪指令的定义

### 数据段



```c
DATAS SEGMENT
    string_1 DB 'Please input a numbers(0-65536):','$'
    string_2 DB 'ERROR: OVERFLOW! Please input again:','$'
    string_3 DB 'The array you have input is:',0ah,0dh,'$'
    string_4 DB 'After Sort the num is:',0ah,0dh,'$'
    string_5 DB ' ','$'
    DATA  DW 10 DUP(?)  
    massege  DB 'The sum of the array is: ',0ah,0DH,'$'   
DATAS ENDS
```

**说明**：

|   string_1   |      输入范围提示      |
| :----------: | :--------------------: |
| **string_2** |    **输入错误提示**    |
| **string_3** |   **输出原数组提示**   |
| **string_4** | **输出排序后数组提示** |
| **string_5** |       **空格符**       |
|   **DATA**   |     **缓冲区数组**     |

<!--more-->

### 堆栈段

```c
STACKS SEGMENT
    DW 256 dup(?)
STACKS ENDS
```

### 代码段

```c
CODES SEGMENT
    ASSUME CS:CODES,DS:DATAS,SS:STACKS
```

## 模块分解与实现

###  DOS输入10个数字

-  输入10个无符号数存入缓冲区，并且保证 $num < 65536$  

为何输入范围是65536呢 一个字的最大表示范围是 $FFFF$ 其在十进制的表示下为 65535

| HEX  |        FFFF         |
| :--: | :-----------------: |
| DEC  |        65535        |
| BIN  | 1111 1111 1111 1111 |

#### 输入函数子程序

```c
;---------输入函数（单数字输入）------------
Input PROC Near
    push AX
    push BX
    push CX
    push DX
;---------输入提示--------------

    MOV BX, 0
    CLC
    MOV DX, 0
;----------输入数字--------------
    Lp_0:
        MOV AH, 1
        INT 21H
        CMP AL, 20H ;回车
        JE L_CRLF

;-----   x belong to [0,9]   ----------        
        SUB AL, 30H ; ASCII -> int
        JL L_ERROR  
        CMP AL, 9
        JG L_ERROR
;-------  string -> int   -----------
        MOV AH, 0   ;将 AL扩展成 AX
        XCHG AX, BX ;保护 AX值
        MOV CX, 10
        MUL CX      ; bx *= 10
        ADD AX , BX
        JC L_ERROR  ; OVERFLOW处理
        XCHG AX, BX 
        JMP Lp_0
    L_ERROR:
        MOV DX, 0
        MOV BX, 0
        CALL CRLF   ; 换行        
        CALL ERROR  ; 输出错误提示
        JMP Lp_0
    L_CRLF:         ; 以换行作为一个数的结束标志
        MOV DX, 0
        MOV DATA[SI], BX ;
        POP DX
        POP CX
        POP BX
        POP AX
        RET
        Input ENDP
```

解析函数功能：

1. 本质类似于高精度计算，将读入的一个串转成数字存储在DATA数组中

2. 分成三大部分
   
   一： 输入提示
   
   二： 错误判断及提示
   
   三： 转化为数字
   
3.  L_ERROR 错误处理

4.  L_CRLF 结束处理

> 我们来举一个$1234$ 的例子
>
> | Register |   1   |  2   |  3   |  4   |
> | :------: | :---: | :--: | :--: | :--: |
> |  **AX**  | **1** |  2   |  3   |  4   |
> |  **BX**  |   0   |  1   |  12  | 123  |
> |  **CX**  |  10   |  10  |  10  |  10  |
>
> $AX + (BX * CX)$

最后将结果存储在DATA数组里

### 实现冒泡排序

冒泡排序作为一个简单的排序算法，时间复杂度    $O(n^2)$ 需要两层循环，为了提高代码的可读性，我们将内层的循环写成一个子程序每次调用

内层循环很简单，每次从头比到尾，遇到比它小的交换就可以了。因为是字操作数，所以循环的下标到18为结束条件。

```c
;---------Bubble_sort--------------------
Bubble_sort PROC NEAR
    PUSH BX
    PUSH DX
    MOV  SI,DI
LOOP1:
    ADD  SI,2  
    MOV  BX,DATA[DI]
    CMP  BX,DATA[SI]
    JA   SWAP
    JMP  NEXT
SWAP:    
    MOV  DX,DATA[SI]
    MOV  DATA[DI],DX
    MOV  DATA[SI],BX 
NEXT:
    CMP SI,18
    JL   LOOP1
    POP  DX
    POP  BX
    RET
    Bubble_sort ENDP 
```

外层调用：每次$DI + 2$

```c
;----------Sort-----------
    MOV CX, 9
    MOV DI, 0
    
FOR1:
    CALL Bubble_sort
    ADD DI, 2
    LOOP FOR1
```

### DOS输出到屏幕

```c
    CALL CRLF
    MOV DX, OFFSET string_4 ;'After Sort the num is:'
    MOV AH, 9
    INT 21H

    MOV CX, 10
    MOV DI, 0
FOR2:
    CALL Print
    CALL Space
    ADD DI , 2
    LOOP FOR2
    CALL CRLF
```

输出DATA内的数字，每次输出一个数字然后在输出一个空格

> Print函数:
>
> 1. 利用DIV函数的特点——每次除10的商放在AX， 余数放入DX
> 2. 并利用栈的 FILO（First in Last Out）的特点
>
> 依旧以1234的例子来看一下是怎么处理的
>
> |     DATA[Num]      | 1234  |   123    |     12      |       1        |
> | :----------------: | :---: | :------: | :---------: | :------------: |
> |       **DX**       | **4** |  **3**   |    **2**    |     **1**      |
> | **Stack(PUSH DX)** | **4** | **4，3** | **4，3，2** | **4，3，2，1** |
> | **Print(POP DX)**  | **4** |  **34**  |   **234**   |    **1234**    |
>
> $\cfrac{DATA[Num]} {10}$ 的余数存入DX

```c
Print PROC Near
    PUSH AX
    PUSH BX
	PUSH CX
	PUSH DX 

    MOV CX, 0
    MOV BX, 10
    MOV AX, DATA[DI]
    LAST:
        MOV DX, 0
        DIV BX ; DIV商放AX，余数放入DX
        PUSH DX
        INC CX
        CMP AX, 0
        JNZ LAST
    AGE:
        POP DX
        OR DX, 30H
        MOV AH, 2
        INT 21H
        LOOP AGE
        POP  DX
        POP  CX
	    POP  BX
	    POP  AX
	    RET
        Print ENDP
```



### 求累加和

- 全部累加到$DATA[0]$ 上直接调用 Print 函数，因为Print函数是针对DATA数组设计的，所以把最后的结果存入DATA数组中不需要额外的输出函数。

```c
;-------SUM-------------
Get_sum PROC NEAR
    PUSH BX
    PUSH CX

    MOV BX, 0
    MOV CX , 9
    MOV DI, 2
LOP1:
    MOV BX, DATA[0]
    ADD BX, DATA[DI]
    MOV DATA[0], BX
    ADD DI , 2
    LOOP LOP1
    POP CX
    POP BX
    RET
    Get_sum ENDP
```

### 其他函数

```c
;----换行子函数（一个数输入完毕）-------
CRLF PROC Near
    push AX
    push DX
    MOV DL, 0ah
    MOV AH, 2
    INT 21H
    pop DX
    pop AX
    RET
    CRLF ENDP
;---------空格-----------
Space PROC Near
    push AX
    push DX
    MOV DX, OFFSET string_5 ;' '
    MOV AH, 9
    INT 21H
    pop DX
    pop AX
    RET 
    Space ENDP
;----------错误提示-------------
ERROR PROC Near
    push BX
    push DX
    MOV DX, OFFSET string_2 ; ERROR: OVERFLOW! Please input again:
    MOV AH, 9
    INT 21H
    pop DX
    pop BX
    RET
    ERROR ENDP
```

## 流程图

#### Input

![](https://gitee.com/relifes/image/raw/master/img/20220305181155.png)

#### Print

![在这里插入图片描述](https://gitee.com/relifes/image/raw/master/img/20220305181223.png)

#### Bubble_Sort

![在这里插入图片描述](https://gitee.com/relifes/image/raw/master/img/20220305181243.png)

#### Get_Sum

![在这里插入图片描述](https://gitee.com/relifes/image/raw/master/img/20220305181258.png)

## 代码与运行截图

### 完整版代码（在MASM运行通过）

```c
;-----数据段------------
DATAS SEGMENT
    string_1 DB 'Please input 10 numbers(0-65536):','$'
    string_2 DB 'ERROR: OVERFLOW! Please input again:','$'
    string_3 DB 'The array you have input is:',0ah,0dh,'$'
    string_4 DB 'After Sort the num is:',0ah,0dh,'$'
    string_5 DB ' ','$'
    DATA  DW 10 DUP(?)  
    massege  DB 'The sum of the array is: ',0ah,0DH,'$'   
DATAS ENDS

;-----堆栈段------------
STACKS SEGMENT
    DW 256 dup(?)
STACKS ENDS

;-----代码段------------
CODES SEGMENT
    ASSUME CS:CODES,DS:DATAS,SS:STACKS


;-----------程序开始------------
START:
    MOV AX,DATAS
    MOV DS,AX
    MOV SI, 0  ;指针初始化
    MOV CX, 10 ;循环次数
;---------Input----------
    MOV DX, OFFSET string_1 ;Please input 10 numbers(0-65536)
    MOV AH, 9
    INT 21H
Lp:
    CALL Input
    ADD SI, 2
    Loop Lp
;--------结束输入，换行---------------
    CALL CRLF
    MOV DX, OFFSET string_3 ;'The array you have input is:'
    MOV AH, 9               ;首地址 DS:DX 
    INT 21H
 ;-------输出 ----------------   
    MOV CX, 10
    MOV DI, 0
Again:
    CALL Print
    CALL Space
    ADD DI , 2
    Loop Again
;/******************************/
;----------Sort-----------
    MOV CX, 9
    MOV DI, 0
    
FOR1:
    CALL Sort
    ADD DI, 2
    LOOP FOR1

    CALL CRLF
    MOV DX, OFFSET string_4 ;'After Sort the num is:'
    MOV AH, 9
    INT 21H

    MOV CX, 10
    MOV DI, 0
FOR2:
    CALL Print
    CALL Space
    ADD DI , 2
    LOOP FOR2
    CALL CRLF
;-------求和输出---------------------    
    MOV DX, OFFSET massege;
    MOV AH, 9
    INT 21H

    CALL Get_sum
    MOV DI, 0
    CALL Print


EXIT:
    MOV AH, 4CH
    INT 21H


;/************子程序调用****************/


;---------输入函数（单数字输入）------------
Input PROC Near
    push AX
    push BX
    push CX
    push DX


    MOV BX, 0
    CLC
    MOV DX, 0
;----------输入数字--------------
    Lp_0:
        MOV AH, 1
        INT 21H
        CMP AL, 20H ;空格
        JE L_CRLF

;-----   x belong to [0,9]   ----------        
        SUB AL, 30H ; ASCII -> int
        JL L_ERROR  
        CMP AL, 9
        JG L_ERROR
;-------  string -> int   -----------
        MOV AH, 0   ;将 AL扩展成 AX
        XCHG AX, BX ;保护 AX值
        MOV CX, 10
        MUL CX      ; bx *= 10
        ADD AX , BX
        JC L_ERROR  ; OVERFLOW处理
        XCHG AX, BX 
        JMP Lp_0
    L_ERROR:
        MOV DX, 0
        MOV BX, 0
        CALL CRLF   ; 换行        
        CALL ERROR  ; 输出错误提示
        JMP Lp_0
    L_CRLF:         ; 以换行作为一个数的结束标志
        MOV DX, 0
        MOV DATA[SI], BX ;
        POP DX
        POP CX
        POP BX
        POP AX
        RET
        Input ENDP


;----换行子函数（一个数输入完毕）-------
CRLF PROC Near
    push AX
    push DX
    MOV DL, 0ah
    MOV AH, 2
    INT 21H
    pop DX
    pop AX
    RET
    CRLF ENDP
;---------空格-----------
Space PROC Near
    push AX
    push DX
    MOV DX, OFFSET string_5 ;' '
    MOV AH, 9
    INT 21H
    pop DX
    pop AX
    RET 
    Space ENDP
;----------错误提示-------------
ERROR PROC Near
    push BX
    push DX
    MOV DX, OFFSET string_2 ; ERROR: OVERFLOW! Please input again:
    MOV AH, 9
    INT 21H
    pop DX
    pop BX
    RET
    ERROR ENDP

;---------输出函数（单数字输出）-------------
Print PROC Near
    PUSH AX
    PUSH BX
	PUSH CX
	PUSH DX 

    MOV CX, 0
    MOV BX, 10
    MOV AX, DATA[DI]
    LAST:
        MOV DX, 0
        DIV BX ; DIV商放AX，余数放入DX
        PUSH DX
        INC CX
        CMP AX, 0
        JNZ LAST
    AGE:
        POP DX
        OR DX, 30H
        MOV AH, 2
        INT 21H
        LOOP AGE
        POP  DX
        POP  CX
	    POP  BX
	    POP  AX
	    RET
        Print ENDP

;---------SORT---------------------
SORT PROC NEAR
    PUSH BX
    PUSH DX
    MOV  SI,DI
LOOP1:
    ADD  SI,2  
    MOV  BX,DATA[DI]
    CMP  BX,DATA[SI]
    JA   CHANGE
    JMP  NEXT
CHANGE:    
    MOV  DX,DATA[SI]
    MOV  DATA[DI],DX
    MOV  DATA[SI],BX 
NEXT:
    CMP SI,18
    JL   LOOP1
    POP  DX
    POP  BX
    RET
    SORT ENDP   
;-------SUM-------------
Get_sum PROC NEAR
    PUSH BX
    PUSH CX

    MOV BX, 0
    MOV CX , 9
    MOV DI, 2
LOP1:
    MOV BX, DATA[0]
    ADD BX, DATA[DI]
    MOV DATA[0], BX
    ADD DI , 2
    LOOP LOP1
    POP CX
    POP BX
    RET
    Get_sum ENDP

CODES ENDS
    END START
```

### 正确运行时截图

![](https://gitee.com/relifes/image/raw/master/img/20220305181322.png)

### 错误输入时截图

![](https://gitee.com/relifes/image/raw/master/img/20220305180619.png)

