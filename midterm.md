# Midterm Review

### String Copy Example (using post-incrementation)

**C**

```C
void str_copy_1(char* src, char* dest, int len) {
    while (len--) {
        *dest++ = *src++;
    }
}
```

**ASM**

```asm
str_cpy_1:
    SUBS R2, R2, #1
    BXLT LR
    LDRB R3, [R0], #1   // Notice that we are incrementing by 1 and not 4 (as usual)
    STRB R3, [R1], #1   // same here
    B str_cpy_1
```

**Things to note:**

* R2 corresponds to len, which is passed by copy (it is not dereferenced in the asm code)
* notice how R0 and R1 are dereferenced in both C and asm code.
* post-incrementation in C translates to post-indexing mode in asm.
* The 'B' is added to LDR and STR to signify we are only loading a single byte (since those are chars)
    * This is also why we only increment to address of the next element by one byte
* `SUBS` is used to update the zero flags when checking at `BXLT`.
* Notice that this is not actually a recursive call (in the asm) since we aren't pushing LR onto the stack.

#### Using the function Example

**C**

```C
#include <stdio.h>

char strng[4] = "abc" // null terminated

void str_copy_1(char* src, char* dest, int len);    // prototype, refer to previous example

int main(void) {
    char arr[4];
    str_cpy_1(strng, arr, 4);   // could use strlen() here, but this adds to the asm code
    return 0;
}
```

**ASM**

```asm
    .text
main:
        SUB SP, SP, #4  // Move stack pointer to allocate memory for the new array
        LDR R0, =strng  // Load string
        MOV R1, SP      // Move the current address of the stack pointer into R1, which is the location of arr
        MOV R2, #4      // len, it's a constant in this case
        PUSH {LR}
        BL str_cpy_1    // function call
        POP {LR}
        ADD SP, SP, #4
        MOV R0, #0
        BX LR

str_cpy_1:
        ...

    .end
    .data
strng:  .byte 0x61, 0x62, 0x63, 0x00    // preloaded in memory since it was initialized
    .end
```

**Things to note:**

* At the start, the address of the stack pointer is 0x00000000, decrementing it makes it wrap around to 0xFFFFFFC
* We load the starting address of strng by reference, using the = symbol
* R0 is the array to copy from, R1 is the empty array, allocated on the stack and R2 is the length of both arrays
* At the end, the `MOV R0, #0` is part of the `return 0;` instruction. Since, by convention, R0 is the return value, we put the error code into it.

### String Copy (using regular indexing)

**C**

```C
void str_copy_1(char* src, char* dest, int len) {
    while (len--) {
        dest[len] = src[len];
    }
}
```

**ASM**

```asm
str_cpy_1:
    SUBS R2, R2, #1
    BXLT LR
    LDRB R3, [R0, R2]   // regular indexing this time, R2 being the offset in bytes.
    STRB R3, [R1, R2]
    B str_cpy_1
```

*pre-indexed mode (useless in this case) looks like: LDRB R3, [R0, R2]!*

**Things to note**

* using array indexing in C directly translates to regular indexing in asm, when post-incrementing the pointer address in C does the exact same thing in asm.
    * C translate almost directly to asm (when no optimizations are used).
* Otherwise, everything works the same, except we fill the array in reverse order (from beginning to end)



