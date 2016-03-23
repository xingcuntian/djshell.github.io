---
layout: post
category: "asm"
title: "asm[第三关]"
tags: ["asm"]
---

### 1.变量赋值
```asm 
.386
.model flat, stdcall

include    windows.inc
include    kernel32.inc
include    masm32.inc
include    debug.inc

includelib kernel32.lib
includelib masm32.lib
includelib debug.lib

.const ;常量
    v1 dd 11
    
.data ;初始化变量
    v2 dd 22
   
    
.data? ;未初始化变量
    v3 dd ?

.code

main proc
	
	PrintDec v1  ;11
	PrintDec v2  ;22
	PrintDec v3  ;0
	
	mov eax, 33	;未初始化的值赋值
	mov v3, eax
	
	PrintDec v1  ;11
	PrintDec v2  ;22
	PrintDec v3  ;33
	
	ret

main endp

end main

```


### 2. 数组赋值：
```asm
.386
.model flat, stdcall


include    windows.inc
include    kernel32.inc
include    masm32.inc
include    debug.inc
includelib kernel32.lib
includelib masm32.lib
includelib debug.lib


.data
	;声明并初始化有三个元素的 DWORD 数组; 该数组每个元素是 4 字节
	val dd 11,22,33
.code
main proc
	
	mov eax, val[4*0]
	PrintDec eax		;11
	
	mov eax, val[4*1]
	PrintDec eax		;22	
	
	
	mov eax, val[4*2]
	PrintDec eax		;33
	
	ret

main endp
end main
```

#### 3. 伪指令 DUP 与数组
```asm
.386
.model flat,stdcall


include windows.inc
include kernel32.inc
include masm32.inc
include debug.inc

includelib kernel32.lib
includelib masm32.lib
includelib debug.lib

.data

    ;声明有三个元素的 DWORD 数组, 并把每个元素初始化为 9
    v1 dd 4 dup(9)

.data?

    ;声明有三个元素的 DWORD 数组, 无初始化; 对全局变量, 没有初始化的将用 0 填充
    v2 dd 3 dup(?)
 
.code

main proc
	
	    DumpMem offset v1, 16  ;09 00 00 00 - 09 00 00 00 - 09 00 00 00 - 09 00 00 00
	    DumpMem offset v2, 12  ;00 00 00 00 - 00 00 00 00 - 00 00 00 00 - 00 00 00 00

	ret

main endp
end main

```

- 请尊重本人劳动成功，可以随意转载但保留以下信息 
- 作者：岁月经年 
- 时间：2016年03月
