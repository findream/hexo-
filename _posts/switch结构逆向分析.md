---
title: switch结构逆向分析
categories: 安全
date: 2017-11-6 17:02:11
tags:
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**暑假看了看雪的知识库，偶然间看到这篇besterChen写的switch结构分析一文，甚喜之，原封不动的抄了下来，在此表示非常感谢。这是在暑假的word版经过复核及重新测试，发现原文许多反汇编代码与原先语句有出入，可能是现在编译器更加高级了。为了保证原文的统一和完整性，在此没有对原文代码进行修改，只是在后面进行了补充，形式为图片类型。随后也会附上关于本次的测试样本（[http://pan.baidu.com/s/1gfo0ejx](http://pan.baidu.com/s/1gfo0ejx)），**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**switch 结构的逆向难度在于不同分支的switch通过反编译器可能演变成不同的汇编语言版本，这就要求逆向工程师精确掌握反汇编switch的规律。灵活的运用调试器进行调试。**
<!-- more -->
# 1.case<=3的情况：
先看第一个程序段：
<pre><code>
    switch (nscore)
    {
    case 1:
        ntmpNum = 1;
        break;
    case 3:
        ntmpNum = 3;
        break;
    case 4:
        ntmpNum = 4;
        break;
    default:
        ntmpNum = 10;
    }
    printf("%d", ntmpNum); // 要调用一下ntmpNum，否则上面的switch会被优化掉
</code></pre>
OD载入，看一下：
<pre><code>
	00401013 >|.  E8 0F010000    call    00401127     ;scanf
	00401018  |.  8B4424 08      mov     eax, dword ptr [esp+8]
	0040101C  |?  83C4 08        add     esp, 8	      ; 上面scanf是C类调用
	0040101F  |?  48             dec     eax	   ; 通过EAX的减法来判断属于哪个分支
	00401020  |?  74 1D          je      short 0040103F
	00401022  |.  83E8 02        sub     eax, 2
	00401025  \.  74 11          je      short 00401038
	00401027      48             dec     eax
	00401028      74 07          je      short 00401031
	0040102A      B8 0A000000    mov     eax, 0A
	0040102F      EB 13          jmp     short 00401044   ;break
	00401031  |.  B8 04000000    mov     eax, 4
	00401036  |.  EB 0C          jmp     short 00401044
    00401038  |?  B8 03000000    mov     eax, 3
    0040103D  |?  EB 05         jmp     short 00401044
	0040103F  |?  B8 01000000    mov     eax, 1
	00401044  |.  50             push    eax
	00401045  |?  68 38904000    push    00409038       ;  ASCII "%d"
</code></pre>
通过上述反汇编代码，我们容易得知：在有规律的switch语句中，汇编代码显得有规律，和if语句一致。测试结果与原文一致。
# 2.case项多于3项且有规律的情况：
第一个程序段：
<pre><code>
    scanf("%d", &nscore);
    switch (nscore)
    {
        case 3:
        	ntmpNum = 1;
       		break;
        case 1:
        	ntmpNum = 3;
        	break;
        case 5:
        	ntmpNum = 4;
        	break;
    	case 9:
        	ntmpNum = 4;
        	break;
    	case 7:
        	ntmpNum = 4;
        	break;
    	case 11:
        	ntmpNum = 4;
        	break;
    	default:
        	ntmpNum = 10;
    }
    printf("%d", ntmpNum); // 要调用一下ntmpNum，否则上面的switch会被优化掉
</code></pre>
这段代码，我们将有规律的case打乱顺序，然后看编译器是怎么处理的。
OD中查看反汇编形式：
<pre><code>
	0040100E  |.  68 38904000   push    00409038            ; ASCII "%d"
	00401013  |.  E8 3F010000   call    00401157            ; scanf
	00401018  |.  8B4C24 08     mov     ecx, dword ptr [esp+8]  ;得到输入的内容
	0040101C  |.  83C4 08       add     esp, 8
	0040101F  |.  8D41 FF       lea     eax, dword ptr [ecx-1]  ;输入的内容-1; Switch (cases 1..B)
	00401022  |.  83F8 0A       cmp     eax, 0A
	00401025  |.  77 1C         ja      short 00401043
	00401027  |.  FF2485 641040>jmp     dword ptr [eax*4+401064] ;查表，跳转到对应的CASE中
	0040102E  |>  B8 01000000   mov     eax, 1             ;  Case 3 of switch 0040101F
	00401033  |.  EB 13         jmp     short 00401048
	00401035  |>  B8 03000000   mov     eax, 3             ;  Case 1 of switch 0040101F
	0040103A  |.  EB 0C         jmp     short 00401048
	0040103C  |>  B8 04000000   mov     eax, 4             ;  Cases 5,7,9,B of switch 0040101F
	00401041  |.  EB 05         jmp     short 00401048
	00401043  |>  B8 0A000000   mov     eax, 0A            ;  Default case of switch 0040101F
	00401048  |>  50            push    eax
	00401049  |.  68 38904000   push    00409038           ;  ASCII "%d"
	0040104E  |.  E8 D3000000   call    00401126           ;  printf
</code></pre>
跟随下这个表，我们发现，这个表就在调用它的函数后，如下：
<pre><code>
	00401064  00401035  switch.00401035
	00401068  00401043  switch.00401043          插入的是default分支的首地址
	0040106C  0040102E  switch.0040102E
	00401070  00401043  switch.00401043          插入的是default分支的首地址
	00401074  0040103C  switch.0040103C
	00401078  00401043  switch.00401043          插入的是default分支的首地址
	0040107C  0040103C  switch.0040103C
	00401080  00401043  switch.00401043          插入的是default分支的首地址
	00401084  0040103C  switch.0040103C
	00401088  00401043  switch.00401043          插入的是default分支的首地址
	0040108C  0040103C  switch.0040103C
</code></pre>
<font color=#DC143C>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;认真对比一下这个表，发现，它先是对case后的常量排序，然后再将对应的处理代码的首地址写成一个表，通过jmp   dword ptr [eax*4+401064] 查表直接进入到对应的case中。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于缺省的case（我们是间隔2递增的case）在表中填充的是default分支的首地址。</font>

* 补充：
    * 我输入的是3，也就是case语句的第二个语句（1.3.5....）
    * 利用IDA pro查看分支表，如图：
    ![](https://i.imgur.com/fGlHnLG.png)
    * 然后查看od跳转的地方（401341）：
    ![](https://i.imgur.com/bdTKhlv.png)
    * 对比od可以知道：401377是默认分支，其他是有效的的case分支。也就是IDA pro显示的分支表中的2.4.6.8....如图，可以知道：在分支表存在的顺序按照nscore排列。
    ![](https://i.imgur.com/EWEajcb.png)

# 3.多于三项部分有规律的情况:
**（1）上个我们发现，它会给缺省的case表项中填补default分支的首地址，那我们将这个间隔调大，观察一下编译器会怎么处理，代码段如下：**
<pre><code>
    scanf("%d", &nscore); 
    switch (nscore)
    {
    	case 1:
        	ntmpNum = 1;
        	break;
    	case 2:
        	ntmpNum = 2;
        	break;
    	case 3:
        	ntmpNum = 3;
        	break;
    //这里丢失20多个case
    	case 26:
        	ntmpNum = 26;
        	break;
    	case 27:
        	ntmpNum = 27;
        	break;
    	case 28:
        	ntmpNum = 28;
        	break;
    	default:
        	ntmpNum = 10;
    }
    printf("%d", ntmpNum); // 要调用一下ntmpNum，否则上面的switch会被优化掉
</code></pre>
反汇编观察一下：
<pre><code>
	0040100E    68 38904000     push    00409038                      ; ASCII "%d"
	00401013    E8 6F010000     call    00401187                      ; scanf
	00401018    8B4C24 08       mov     ecx, dword ptr [esp+8]        ; 得到输入的内容
	0040101C    83C4 08         add     esp, 8
	0040101F    8D41 FF         lea     eax, dword ptr [ecx-1]        ; 输入的内容-1
	00401022    83F8 1B         cmp     eax, 1B
	00401025    77 39           ja      short 00401060
	00401027    33D2            xor     edx, edx
	00401029    8A90 9C104000   mov     dl, byte ptr [eax+40109C]     ; 检索case的下标索引值表
	;它参与运算从地址表中找到对应的case地址
	0040102F    FF2495 80104000 jmp     dword ptr [edx*4+401080]  
	; 通过值表填充的CASE索引值，查地址表
	00401036    B8 01000000     mov     eax, 1
	0040103B    EB 28           jmp     short 00401065			      ; break
	0040103D    B8 02000000     mov     eax, 2
	00401042    EB 21           jmp     short 00401065
	00401044    B8 03000000     mov     eax, 3
	00401049    EB 1A           jmp     short 00401065
	0040104B    B8 1A000000     mov     eax, 1A
	00401050    EB 13           jmp     short 00401065
	00401052    B8 1B000000     mov     eax, 1B
	00401057    EB 0C           jmp     short 00401065
	00401059    B8 1C000000     mov     eax, 1C
	0040105E    EB 05           jmp     short 00401065
	00401060    B8 0A000000     mov     eax, 0A
	00401065    50              push    eax
	00401066    68 38904000     push    00409038                         ; ASCII "%d"
</code></pre>
下标索引表：
<pre><code>
	0040109C    00          DB 00	case 1的索引值
	0040109D    01          DB 01	case 2的索引值
	0040109E    02          DB 02	case 3的索引值
	0040109F    06          DB 06	下面全部填充default的索引值
	004010A0    06          DB 06
	...
	004010B4    06          DB 06
	004010B5    03          DB 03	case 4的索引值
	004010B6    04          DB 04	case 5的索引值
	004010B7    05          DB 05	case 6的索引值
</code></pre>
这样查两个表，缺省的case项在索引表中插入 default 的索引值，这样每个case项就节省了3个字节的空间。
<pre><code>
	mov     dl, byte ptr [eax+40109C]   // 40109C是索引表首地址
	jmp     dword ptr [edx*4+401080]    // 401080是跳转地址表的首地址。
</code></pre>
* 补充：
    * 根据OD和IDA的联合调试，发现这个和上面的方法是一样的。直接使用的是地址值，并没有经过索引表来中介传输。
	![](https://i.imgur.com/9tMLKKV.png)
	![](https://i.imgur.com/s6wouQW.png)
**（2）我们继续增大这个case之间的差距，让它超过255，代码段如下：**
<pre><code>
    scanf("%d", &nscore)；
    switch (nscore)
    {
    	case 1:
        	ntmpNum = 1;
        	break;
    	case 2:
        	ntmpNum = 2;
        	break;
    	case 3:
        	ntmpNum = 3;
        	break;
    	//这里丢失几个case
    	case 326:
        	ntmpNum = 26;
        	break;
    	case 327:
        	ntmpNum = 27;
        	break;
    	case 328:
        	ntmpNum = 28;
        	break;
    	default:
        	ntmpNum = 10;
    }
    printf("%d", ntmpNum); // 要调用一下ntmpNum，否则上面的switch会被优化掉
</code></pre>
反汇编看一下效果：
<pre><code>
	00401018    8B4424 08       mov     eax, dword ptr [esp+8]  ; 得到输入的内容
	0040101C    83C4 08         add     esp, 8
	0040101F    3D 46010000     cmp     eax, 146            ; 判断是不是大case中最小的
	00401024    7F 27           jg      short 0040104D	; 如果大于，就进入大case中比较
	00401026    74 1E           je      short 00401046	; 如果相等就直接进入0x146的case代码段
	00401028    48              dec     eax			; 否则就到小的case段中比较。
	00401029    74 14           je      short 0040103F
	0040102B    48              dec     eax
	0040102C    74 0A           je      short 00401038
	0040102E    48              dec     eax
	0040102F    75 26           jnz     short 00401057	; default了。
	00401031    B8 03000000     mov     eax, 3
	00401036    EB 32           jmp     short 0040106A
	00401038    B8 02000000     mov     eax, 2
	0040103D    EB 2B           jmp     short 0040106A
	0040103F    B8 01000000     mov     eax, 1
	00401044    EB 24           jmp     short 0040106A
	00401046    B8 1A000000     mov     eax, 1A
	0040104B    EB 1D           jmp     short 0040106A
	0040104D    2D 47010000     sub     eax, 147	
	; 减去一个case项值，得到一个差值，这样就可以判断大case了。
	00401052    74 11           je      short 00401065
	00401054    48              dec     eax
	00401055    74 07           je      short 0040105E
	00401057    B8 0A000000     mov     eax, 0A
	0040105C    EB 0C           jmp     short 0040106A
	0040105E    B8 1C000000     mov     eax, 1C
	00401063    EB 05           jmp     short 0040106A
	00401065    B8 1B000000     mov     eax, 1B
	0040106A    50              push    eax
	0040106B    68 38904000     push    00409038            ; ASCII "%d"
	00401070    E8 B1000000     call    00401126
</code></pre>
看到了么？这里就分成了两段，每段当做if来处理的，我想应该是我们每段的case数量太少，我们让上面的case 数量大于3个试试，看看会不会是只要大于三项的有规律case就查表，少于等于3项的就当成if来处理。

* 补充：
    * 我也不清楚为什么编译器会和146.147.148这三个数比较，然后跳入第二次default。
    ![](https://i.imgur.com/BPjANEv.png)

代码如下：
<pre><code>
    scanf("%d", &nscore);
    switch (nscore)
    {
    	case 1:
        	ntmpNum = 1;
        	break;
    	case 2:
        	ntmpNum = 2;
        	break;
    	case 3:
        	ntmpNum = 3;
        	break;
    	case 4:
        	ntmpNum = 4;
        	break;
    	case 5:
        	ntmpNum = 5;
        	break;
    	//这里丢失几个case
    	case 326:
        	ntmpNum = 326;
        	break;
    	case 327:
        	ntmpNum = 327;
        	break;
    	case 328:
        	ntmpNum = 328;
        	break;
    	default:
    	    ntmpNum = 10;
    }
	printf("%d", ntmpNum); // 要调用一下ntmpNum，否则上面的switch会被优化掉
</code></pre>
反汇编看下效果：
<pre><code>
	0040100E    68 38904000     push    00409038                         ; ASCII "%d"
	00401013    E8 5F010000     call    00401177                         ; scanf
	00401018    8B4424 08       mov     eax, dword ptr [esp+8]           ; 得到输入的内容
	0040101C    83C4 08         add     esp, 8
	0040101F    3D 46010000     cmp     eax, 146                      
	00401024    7F 39           jg      short 0040105F
	00401026    74 30           je      short 00401058
	00401028    48              dec     eax
	00401029    83F8 04         cmp     eax, 4
	0040102C    77 3B           ja      short 00401069
	0040102E    FF2485 98104000 jmp     dword ptr [eax*4+401098]
	00401035    B8 01000000     mov     eax, 1
	0040103A    EB 40           jmp     short 0040107C
	0040103C    B8 02000000     mov     eax, 2
	00401041    EB 39           jmp     short 0040107C	
	00401043    B8 03000000     mov     eax, 3
	00401048    EB 32           jmp     short 0040107C
	0040104A    B8 04000000     mov     eax, 4
	0040104F    EB 2B           jmp     short 0040107C
	00401051    B8 05000000     mov     eax, 5
	00401056    EB 24           jmp     short 0040107C
	00401058    B8 46010000     mov     eax, 146
	0040105D    EB 1D           jmp     short 0040107C
	0040105F    2D 47010000     sub     eax, 147
	00401064    74 11           je      short 00401077
	00401066    48              dec     eax
	00401067    74 07           je      short 00401070
	00401069    B8 0A000000     mov     eax, 0A
	0040106E    EB 0C           jmp     short 0040107C
	00401070    B8 48010000     mov     eax, 148
	00401075    EB 05           jmp     short 0040107C
	00401077    B8 47010000     mov     eax, 147
	0040107C    50              push    eax
	0040107D    68 38904000     push    00409038                         ; ASCII "%d"
	00401082    E8 BF000000     call    00401146
</code></pre>
跳转表如下：
<pre><code>
	00401098  00401035  switch.00401035	
	0040109C  0040103C  switch.0040103C
	004010A0  00401043  switch.00401043
	004010A4  0040104A  switch.0040104A
	004010A8  00401051  switch.00401051
</code></pre>
哈哈，不多说了，我们看下无规律的情况。
* 补充（原文的反汇编代码和这次实验不一样，原文存在地址表，这个不存在）
     * 可能是利用最优二叉树算法得到的中间的146.147.148的值。
     ![](https://i.imgur.com/NuuD0FS.png)
     ![](https://i.imgur.com/T7M4S7F.png)
     ![](https://i.imgur.com/ViSkkB0.png)
# 4.对于毫无规律的情况。
**(1)通过上个例子的分析，我们大概可以猜出来，编译器会择优选择查表，查双表来对部分离得比较近的case项作处理，最后才考虑毫无规律的情况，为了提高我们这次测试的成功率，我们让每个相邻的case项差值都超过255，为了避免switch当做if来处理，我们多写几个case，具体代码段如下:**
<pre><code>
    scanf("%d", &nscore); 
    switch (nscore)
    {
    	case 1:
    	    ntmpNum = 1;
    	    break;
    	case 300:
    	    ntmpNum = 300;
    	    break;
    	case 570:
    	    ntmpNum = 570;
    	    break;
    	case 830:
    	    ntmpNum = 830;
    	    break;
    	case 1094:
    	    ntmpNum = 1094;
    	    break;
    	case 1314:
    	    ntmpNum = 32;
    	    break;
    	case 1614:
    	    ntmpNum = 1614;
    	    break;
   		case 1894:
    	    ntmpNum = 1894;
    	    break;
    	case 2199:
    	    ntmpNum = 2199;
    	    break;
   		case 2578:
  	        ntmpNum = 2578;
            break;
    	case 2800:
        	ntmpNum = 2800;
        	break;
    	case 3178:
        	ntmpNum = 3178;
        	break;
    	case 3568:
        	ntmpNum = 3568;
        	break;
    	case 3856:
        	ntmpNum = 3856;
        	break;
    	case 4212:
        	ntmpNum = 4212;
        	break;
    	case 4679:
        	ntmpNum = 4679;
        	break;
    	case 5050:
        	ntmpNum = 5050;
        	break;
        case 5486:
        	ntmpNum = 5486;
        	break;
    	case 5797:
        	ntmpNum = 5797;
        	break;
    	case 6089:
        	ntmpNum = 6089;
        	break;
    	case 6713:
        	ntmpNum = 6713;
        	break;
    	case 8425:
    	    ntmpNum = 8425;
    	    break;
    	case 8973:
        	ntmpNum = 8973;
        	break;
    	case 9545:
        	ntmpNum = 9545;
        	break;
    	case 9987:
        	ntmpNum = 9987;
        	break;
    	case 11254:
        	ntmpNum = 11254;
        	break;
    	case 12489:
        	ntmpNum = 12489;
        	break;
    	case 15798:
        	ntmpNum = 15798;
        	break;
    	case 26874:
        	ntmpNum = 26874;
        	break;
    	case 34721:
        	ntmpNum = 34721;
        	break;
    	case 39681:
        	ntmpNum = 39681;
        	break;
    	default:
        	ntmpNum = 10;
    }
    printf("%d", ntmpNum); // 要调用一下ntmpNum，否则上面的switch会被优化掉
</code></pre>
反汇编结果和上述理论一致。
