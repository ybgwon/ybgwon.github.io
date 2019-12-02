+++
title = "Kernel-Study-2019-11-16"
author = ["Yong-Beom Gwon"]
date = 2019-11-16T15:40:00+09:00
draft = false
+++

## 주요 코드 {#주요-코드}

{{< highlight C "linenos=table, linenostart=1" >}}
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
{{< /highlight >}}


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
