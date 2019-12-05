+++
title = "Kernel-Study-2019-11-16"
author = ["Yong-Beom Gwon"]
date = 2019-11-16T15:40:00+09:00
draft = false
+++

## 주요 코드 {#주요-코드}

```C
static inline void arch_local_irq_disable(void)
{
      asm volatile(ALTERNATIVE(
	      "msr	daifset, #2		// arch_local_irq_disable",
	      "msr_s  " __stringify(SYS_ICC_PMR_EL1) ", %0",
	      ARM64_HAS_IRQ_PRIO_MASKING)
	      :
	      : "r" ((unsigned long) GIC_PRIO_IRQOFF)
	      : "memory");
}
```


## DAIF 레지스터 {#daif-레지스터}

Interrupt Mask Bit. interrupt mask bits에 억세스 할 수 있도록 한다.  
ARMv8-A\_Architecture\_Reference\_Manual\_(Issue\_A.a) - C4.3.2  

{{< figure src="/images/daif-1.png" link="/images/daif-1.png" >}}  

{{< figure src="/images/daif-2.png" link="/images/daif-2.png" >}}  


### PSTATE fields 접근 명령 {#pstate-fields-접근-명령}

-   daifset  
    
    ```text
    MSR DAIFSet, #Imm4    ; Used to set any or all of DAIF to 1
    ```


## ALTERNATIVE 매크로 {#alternative-매크로}

ARM에서는 ALTERNATIVE 기능이 적용되지 않았다.  
ARM64의 경우 ARMv8.1 코드와 ARMv8.2 코드를 준비해두고 부트업시  
cpu capabilities(features)를 확인하여 가능하면 성능이 더 좋은  
ARMv8.2 코드로 replacement 하도록 사용한다.  


## asm volatile 사용법 {#asm-volatile-사용법}

kernel 코드에서 자주 나오는 inline asembly이다  

```c
__asm__ __volatile__ (asms : output : input : clobber);  
```

-   asm  
    다음에 나오는 것이 인라인 어샘블리 임을 나타낸다
-   volatile  
    최적화를 하지 않는다.
-   asms  
    따옴표로 둘러싸인 어셈블리 코드. %x 와 같은 형태로 input, output  
    파라메터 사용

-   output  
    결과값을 출력하는 변수. 쉼표로 구분
-   input  
    인라인 어셈블리 코드에 넘겨주는 파라메터
-   clobber  
    값이 바뀌는 레지스터. 각 변수는 쉼표로 구분되고 각각을 따옴표로 감싼다.

output input clobber 는 마지막이 없으면 앞의 ':'를 생략할 수있으나 중간에 것이 없으면 반드시 콜론을 표기 해야 한다.  

```asm
#include <stdio.h>

int main(void)
{
      unsigned long a = 5;
      unsigned long b = 10;
      unsigned long sum = 0;

      __asm__("addq %1,%2" : "=r" (sum) : "r" (a), "0" (b));
      printf("a + b = %lu\n", sum);
      return 0;
}
```

\_\_asm\_\_("addq %1,%2" : "=r" (sum) : "r" (a), "0" (b));  
위 코드에서 asm 부분만 따로 떼어 내서 분석해 보면  


### 어셈블리 코드 {#어셈블리-코드}

"addq %1,%2"  
%1,%2 는 output operands 부터 좌측에서 순서대로 0번 부터 매겨지는레지스터 변수의 순서이다.  
위의 경우는 sum 부분의 레지스터가 0번이다. 그리고 a는 1, b는 2 번순으로 된다. b의 경우는 다시 "0"으로 표기하여 output 의 0번째  
sum 부분 레지스터를 사용하게 되어 sum 부분 레지스터는 재사용하게 된다.  


### Input  Operands(입력 인수) & Output Operands(출력 인수) {#input-operands--입력-인수--and-output-operands--출력-인수}

-   형식  
    
    ```text
    [ [asmSymbolicName] ] constraint (C-expression)
    ```


#### asmSymbolicName {#asmsymbolicname}

생략 가능하고 지정하는 경우 %0, %1, …과 같이 인수의 순번을사용하는 대신 심볼명을 사용할 수도 있다.  

```asm
asm (“add %[tmp], #2” : [tmp] “=r” (tmp));
```


#### constraint {#constraint}

"=r" 또는 "0"  

`=r` 에서 `=` 은 modifier 가 되고 r이 일반 레지스터를나타내는 contraint 이다.  

1.  'r' - 일반 레지스터
2.  'm' - 유효 메모리 주소
3.  '[digits]' - Output Operands에서 숫자 번째 항목


#### modifier {#modifier}

1.  '=' - 쓰기 전용
2.  '+' - 읽기 쓰기
3.  '&' - OutputOperands에서 레지스터 할당 순서를 먼저 할 수있도록 요청.  
    보통 input operands에 레지스터를 할당하고 그 후  
    output operands의 레지스터를 사용하기 때문에 input operands에서사용했던 레지스터를 output operands 레지스터로 배치하는경우도 생기는데 그러면서 문제가 될 수 있는 곳에 사용


## 참조 {#참조}

[kldp](https://wiki.kldp.org/KoreanDoc/html/EmbeddedKernel-KLDP/app3.basic.html)  
[linux inside](https://0xax.gitbooks.io/linux-insides/content/Theory/linux-theory-3.html?q=)  
<http://jake.dothome.co.kr/inline-assembly/>
