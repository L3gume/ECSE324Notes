# Subroutines

* We should be able to *call* a subroutine from anywhere in the program, i.e. *change the PC* so that the routine is executed.
* A subroutine must be able to *return* from subroutine, i.e. change the PC so that the execution continues immediately after the point where it was called.
* We should be able to pass params (or args) that may take different values across different calls.
* A subroutine must be able to *return* a value.

## Calling and Return

```C
boo() {
    coo()
    ...
    return;
}

coo() {
    return;
}
```

Would translate into:

```asm
boo:
        BL  coo // LR <- [PC] + 4
                // PC <- coo
        ...
coo:
        BX  LR  // PC <- [LR]
```

The ARM subroutine *call* is implemented with the "Branch and Link" instruction `BL` that stores the address of the next instruction (return address) in the *link register* `LR (R14)`.

To *return from subroutine*, we branch to the address stored in the link register with the `BX` instruction (you can only branch to a label with B, and to a pointer with `BX`).

### Multiple nested calls

```C
boo() {
    coo();
    doo();  //B1
    return; //B2
}

coo() {
    doo();
    return; //C
}

doo() {
    return;
}
```

boo calls coo -> save B1  
coo calls doo -> save C  
doo returns to coo -> C -> PC  
coo returns to boo -> B1 -> PC  
boo calls doo -> save B2  
doo returns to boo -> B2 -> PC  

## Stacks

### Stacks in ARM

* Stacks are used to support subroutines, so ARM has hardware support for a processor stack.
* The data element in the processor stack will always be a word.
* The bottom of stack is at a fixed address and the top of stack grows "upward", towards lower memory addresses.
* Register `R13` is used as a *stack pointer* to point to TOS, also called SP.

### Stack operations

Push from `Rj`:

> STR   Rj, [SP, #-4]!

Pop into `Rj`:

> LDR   Rj, [SP], #4

Peek(i) into `Rj`:

> LDR   Rj, [SP,#const] // where const = 4*i

### Pushing and Poping Multiple Elements

> PUSH {R1, R3-R5}

is a pseudoinstruction equivalent to:

> STMDB     SP!, {R1, R3-R5}

> POP {R1, R3-R5}

is a pseudoinstruction equivalent to:

> LDMIA     SP!, {R1, R3-R5}

### Multiple Nested Calls

If you are a subroutine that will call another subroutine, follow this convention:

**Before calling anybody**  
Push the return address stored in LR on the stack

**When done calling**  
Pop the return address off the stack into LR

*Nice table in the slides*

### Passing Parameters and Return Values

For a small number of parameters you can use the ARM *calling convention*: use R0-R3 for passing parameters, and use R0 for the return value.

*example:*

```asm
        MOV         R0, #1
        MOV         R1, #2
        MOV         R2, #3
        STR         LR, [sp, #-4]!  // save return address
        BL          add3
        STR         R0, SUM         // return value is in R0
        LDR         LR, [sp], #4    // restore return address
stop:   B           stop

        ...

add3:   ADD         R0, R0, R1;
        ADD         R0, R0, R2;
        BX          LR
```

### Callee-save

* In the prev example, the *callee* overwrote R0, which was Ok, since the *caller* knew that the return value would be in R0.
* In general, the caller may need the register values after the callee returns, so the rule is a *callee is responsible for leaving the registers as it found them.*
* *Callee-save convention:* a subroutine should save any registers it wants to use on the stack and the restore the original values to the registers after it is finished using them.

### Passing Parameters on the Stack

Passing params by register is faster. if passing by registers, save the registers on the stack before using them for parameters.

*Program to sum a list of numbers:*

```asm
            .test
            .global_start
_start:     LDR     R0, =ARRAY      // R0 points to ARRAY
            LDR     R1, N           // R1 contains the number of elements to add
            PUSH    {R0, R1, LR}    // push parameters and LR
            BL      listadd         // call subroutine
            LDR     R0, [SP, #4]    // get return value from stack
            STR     R0, SUM         // store into memory
            LDR     LR, [SP, #8]    // restore LR
            ADD     SP, SP, #12     // remove params from stack
stop:       B       stop

listadd:    PUSH    {R0-R3}         // callee-save the registers listadd will use
            LDR     R1, [SP, #20]   // load param N from stack
            LDR     R2, [SP, #16]   // load param ARRAY from stack
            MOV     R0, #0          // clear R0 (sum)
loop:       LDR     R3, [R2], #4    // get next value from ARRAY
            ADD     R0, R0, R3      // form the partial sum
            SUBS    R1, R1, #1      // decrement loop counter
            BGT     loop
            STR     R0, [SP, #20]   // store sum on stack, replacing N
            POP     {R0-R3}         // restore registers
            BX      LR

ARRAY:      .word   6,5,4,3,2,1,14,13,12,11,10,9,8,7
N:          .word   14
SUM         .space  4
            .end
```

### Passing by Value/Reference

```asm
LDR     R0, =ARRAY  // R0 points to ARRAY
LDR     R1, N       // R1 contains the number of scores
```

* The parameter N was *Passed by value*, the acutal value of N (14) was passed to the subroutine.
* The parameter ARRAY was *passed by reference*, a pointer to the first element of the array was passed.

### Stack Frames

* The subroutine can also allocate local variables, only accessible by the subroutine, on the stack
* Using a fram pointer (usually R11) gives a consistent reference to parameters [FP, #const] and local variables [FP, #-const], which moves around relative to the SP.
* When nesting, the stack frame also includes the return address.
