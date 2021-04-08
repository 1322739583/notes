# 汇编学习 [BX]和loop指令

## [bx]的使用

[bx]表示一个物理地址，而且只能用bx。不能用其它的通用寄存器。

```assembly
mov ax,100h  
mov ds,ax
mov ax,[bx] 
```



bx里面存偏移地址，[bx]=ds*16+bx。[bx]就是cs:ip的另一种表示形式。

上面代码的执行到最后一步的时候，寄存器的状况如图所示。

![666](https://pics-note.oss-cn-hangzhou.aliyuncs.com/asm/bx_ax_value.png)

这里[bx]=ds*16+bx=1000h，因为bx=0。所以[bx]表示1000h这个内存地址。而1000h这个地址的内容是B8,也就是第一条指令mov ax,100h.的mov操作码。所以最后ax的值b8也就不奇怪了。



## loop的使用

在来看下面这个loop的例子

例一：123*236

```assembly
assume cs:code 
code segment
    
    mov ax,0
    mov cx,236
    
    s:add ax,123
    
    loop s  
    
   mov ax,4c00h
   int 21h
    
code ends
end
```



汇编实现loop的原理就是通过cx来实现计数，cx也叫计数寄存器（count reg）。loop s会跳转到s的地方，同时cx-1.当cx=0使结束循环。

通过下面这两行中断代码来结束循环。如果没有这个中断的话，程序会报the emulator is halted错误。意思是模拟器奔溃了。

mov ax,4c00h
int 21h

例二：2^10



## loop和[bx]的结合

**计算ffff:0到ffff:3内容的和**:

错误例子：

```assembly
assume cs:code
code segment
	mov ax,ffffh;错误一 
	mov ds,ax
	mov ax,ds:[0];错误二
            
code ends
end
```

> 错误一：数前面必须有0，不能为字母

第一行 mov,0ffffh必须加0前缀，不然会报错：`probably no zero prefix for hex; or no 'h' suffix; or wrong addressing; or undefined var: ffffh `

> 错误二：ds:[0]表示的是8位的内存单元，而ax是16位的寄存器，相加会出现截取的情况

这里ax的值为ffff,ds:[0]也就是内存ffff0的值为ff(这个在内存中好像是固定的)，相加结果为fffe,相对于al+ff=1fe。高位1被截取了，al的结果就变成了fe,ah的值不变，还是ff,所以结果为fffe。

所以要想实现这种地址单元的累加功能必须需要两个寄存器，一个存结果一个存变量。

下面的代码非常的好理解，**最核心的内容就是ds:[0]表示的是8位的数。**

```assembly
assume cs:code
code segment
mov ax,0ffffh
mov ds,ax  
mov dx,0 

mov ah,0 
mov al,ds:[0]
add dx,ax    

mov ah,0 
mov al,ds:[1]
add dx,ax  

mov ah,0 
mov al,ds:[2]
add dx,ax     

mov ah,0 
mov al,ds:[3]
add dx,ax 

            
code ends
end
```

这样就实现了内存单元内容的累加，但是非常的麻烦，这里只是计算4个单元。所以我们可以结合loop简化代码。

```assembly
mov ax,0ffffh
mov ds,ax  
mov bx,0
mov dx,0 
mov cx,4
s:
mov ah,0
mov al,ds[bx]
add dx,ax
inc bx
loop s
```













 