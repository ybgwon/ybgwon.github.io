+++
title = "kernel-study-2019-11-23"
author = ["Yong-Beom Gwon"]
date = 2019-12-05T19:37:00+09:00
draft = false
+++

boot\_cpu\_init 함수 분석. 첫번째 processor 를 활성화한다.  

```c
void __init boot_cpu_init(void)
{
      int cpu = smp_processor_id();

      /* Mark the boot cpu "present", "online" etc for SMP and UP case */
      set_cpu_online(cpu, true);
      set_cpu_active(cpu, true);
      set_cpu_present(cpu, true);
      set_cpu_possible(cpu, true);

#ifdef CONFIG_SMP
      __boot_cpu_id = cpu;
#endif
}
```


## smp\_processor\_id {#smp-processor-id}

percpu 변수 cpu\_number 값을 매크로 함수를 통해 읽어 온다  


### cpu\_number {#cpu-number}

```c
#define raw_smp_processor_id() (*raw_cpu_ptr(&cpu_number))
```

raw\_cpu\_ptr 매크로 함수로 넘어가는 cpu\_number 파라메터는아래와 같이 정의돼 있다.  

```c
DECLARE_PER_CPU_READ_MOSTLY(int, cpu_number);
```

결국 아래와 같이 변환된다.  

```c
extern __attribute__((section(.data..percpu..read_mostly)) int cpu_number
```


#### `__typeof__` {#typeof}

변수 타입 반환 gcc 확장기능  

```c
int i;
__typeof__(i) j;
/* above equals int j */
```


### raw\_cpu\_ptr {#raw-cpu-ptr}

매크로 함수  

```c
#define raw_cpu_ptr(ptr)						\
({									\
      __verify_pcpu_ptr(ptr);						\
      arch_raw_cpu_ptr(ptr);						\
})
```


#### \_\_verify\_pcpu\_ptr(ptr) {#verify-pcpu-ptr--ptr}

\_\_CHECKER\_\_(정적분석도구) 가 define 되어 있을때 ptr이 percpu 영역에 있는지 검사  


#### arch\_raw\_cpu\_ptr(ptr); {#arch-raw-cpu-ptr--ptr}

ptr 과 \_\_my\_cpu\_offset를 매개 변수로 RELOC\_HIDE 호출  

```c
#define SHIFT_PERCPU_PTR(__p, __offset)									\
      RELOC_HIDE((typeof(*(__p)) __kernel __force *)(__p), (__offset))
```

-   RELOC\_HIDE - 컴파일러 에러 방지 위해 사용하는 매크로 함수.  
    <http://studyfoss.egloos.com/5374731>

-   return 값  
    
    ```c
    #define RELOC_HIDE(ptr, off)        \
    ({									\
        unsigned long __ptr;						\
        __asm__ ("" : "=r"(__ptr) : "0"(ptr));				\
        (typeof(ptr)) (__ptr + (off));					\
    })
    ```
    
    마지막 \_\_ptr + (off) return. 마지막 부분은 선언만 하고 있으나  
    gcc 확장기능에 의해 return 된다  
    gcc 확장기능 - compound statement(중괄호로 둘러싼 여러 statement)를  
    expression으로 해석하는 기능.  
    최종 결과는 아래와 같다.  
    
    ```c
    (int *) (&cpu_number  + tpidr_el1 value)       
    ```


## set\_cpu\_online(set\_cpu\_active,set\_cpu\_present,set\_cpu\_possible) {#set-cpu-online--set-cpu-active-set-cpu-present-set-cpu-possible}

struct cpumask 형식의 \_\_cpu\_online\_mask  
구조체의 멤버인 bits 배열에 현재 cpu 의 bit를 set 한다.  

```c
struct cpumask __cpu_online_mask { unsigned long bits[4]; };
cpumask_set_cpu(cpu, &__cpu_online_mask);
/*
 * cpu 가 129 라면 129/64 0~63, 64~127, 128,129... 순이 되고
 * 3번째 인덱스의 2번째 비트를 1로 set하게 되므로 bits[2]
 * 의 값이 2 가 된다. bits[2] 이 원래 0 이었을 경우
 */
bits[2] == 2;
```


### set\_bit(cpumask\_check(cpu), cpumask\_bits(dstp)) {#set-bit--cpumask-checkcpu--cpumask-bits--dstp}

setbit 함수의 경우 현재 cpu 변수와 cpumask 구조체의 포인터를가지고 결국 bits 배열에 cpu 변수를 set 하는 atomic asm 명령어를호출하게 된다.  

```c
static inline void arch_atomic64_##op(long i, atomic64_t *v)		\
{									\
      register long x0 asm ("x0") = i;				\
      register atomic64_t *x1 asm ("x1") = v;				\
								      \
      asm volatile(ARM64_LSE_ATOMIC_INSN(__LL_SC_ATOMIC64(op),	\
"	" #asm_op "	%[i], %[v]\n")					\
      : [i] "+r" (x0), [v] "+Q" (v->counter)				\
      : "r" (x1)							\
      : __LL_SC_CLOBBERS);						\
}

ATOMIC64_OP(or, stset)
```

위 inline asm 명령은 lse 기능(ARMV8.1이상)을 cpu 에서지원하느냐에 따라 LL/SC 방식이나 CAS 구현방식을 사용하는 LSE 중에  
ALTERNATIVE 매크로를 이용하여 atomic 명령을 선택하게 된다.  

{{< figure src="/images/stset.png" link="/images/stset.png" >}}  


### Atomic Operation {#atomic-operation}

1.  ARMv5 이하모두 UP(Uni Processor) 시스템용으로만 구현되어 있어 인터럽트를 막는 것으로 구현
2.  ARMv6, ARMv7  
    UP로 커널을 빌드하여 사용 시 인터럽트를 막는 것으로 구현  
    SMP로 커널을 빌드하여 사용 시 LL/SC 방식으로 구현
3.  ARMv8  
    UP/SMP 가리지 않고 LL/SC 방식으로 구현
4.  ARMv8.1~  
    UP/SMP 가리지 않고 CAS 방식(LSE atomic 명령 채용)으로 구현


### 참조 {#참조}

<http://jake.dothome.co.kr/atomic/>
