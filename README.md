# DECA_Section2_Challenge
Challenge work for DECA Spring Term Section 2

---

### Table of Contents

 1. [Background](#background)
 2. [Implimentation](#implimentation)
 3. [Coming up with new instructions](#coming-up-with-new-instructions)

---
### Background:
Speeding up multiplication in the EEP1 needs definitions for new ALU instructions. This is possible as some combinations of bits are not yet currently used such as in the MOV instruction

There are 7 additional MOV instructions, using machine code for the MOV instructions with non-zero in the c register field INS(4:2). The assembler recognises these as instructions MOVCn Ra Rb where n is a number from 1-7 and will correctly produce the right machine code. For example MOVC1 Ra Rb, could peform the action Ra := Ra op Rb where op is some operation we must define.

Speeding up multiplication should be done by reducing the number of cycles that need to be run in order to have the same output as before. This could be done by merging different stages of the asssembly code into one instruction.

---

### Implimentation:
In order to impliment this there are 4 different stages we will have to complete which I will keep a track of throughout to make it clearer to understand where we are in the completion of this task.

These stages are:

1) Coming up with new instructions to speed up the multiplication
2) Creating the hardware to impliment the new instruction
3) Writing new assembly code using the new instruction word
4) Testing the new software and hardware to ensure the correct output happens for all edge cases

I will keep track of these through a checklist which I will display at the end of each new task:

- [ ] Come up with new instruction
- [ ] Impliment Hardware
- [ ] Create new assembly code
- [ ] Test the new software and hardware together

---

### Coming up with new instructions:

The original c++ code for assembly looked like this:

```cpp
unsigned int op1, op2, op2_shifted, sum;

sum = 0;
op2_shifted = op2;

while(op1 != 0){
 if(op1 & 1){
  sum = sum + op2_shifted;
 }
 op2_shifted = op2_shifted << 1;
 sum = sum >> 1;
}
```
