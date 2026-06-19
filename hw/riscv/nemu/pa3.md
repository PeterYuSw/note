# PA3

## 1. exception
- process

```C
// 1. init mtvec
cte_init
    - asm volatile("csrw mtvec, %0" : : "r"(__am_asm_trap));

// 2. ecall
yield
    - asm volatile("li a7, -1; ecall");
        - isa_raise_intr(word_t NO, vaddr_t epc)
            // set mcause
            - mcause = NO
            // set mepc
            - mepc = s->pc + 4
            - s->dnpc = mtvec // __am_asm_trap

// 3. trap handler
__am_asm_trap: // trap.S
    - call __am_irq_handle
    - mret
        - s->dnpc = mepc

```
