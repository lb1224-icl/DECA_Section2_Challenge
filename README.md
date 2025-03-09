# DECA_Section2_Challenge
Challenge work for DECA Spring Term Section 2

---

### Table of Contents

 1. [Background](#background)
 2. [Implimentation](#implimentation)
 3. [Coming up with new instructions](#coming-up-with-new-instructions)
 4. [Creating the hardware](#creating-the-hardware)

---
### Background:
Speeding up multiplication in the EEP1 needs definitions for new ALU instructions. This is possible as some combinations of bits are not yet currently used such as in the MOV instruction

There are 7 additional MOV instructions, using machine code for the MOV instructions with non-zero in the c register field INS(4:2). The assembler recognises these as instructions MOVCn Ra Rb where n is a number from 1-7 and will correctly produce the right machine code. For example ```MOVC1 Ra Rb```, could peform the action Ra := Ra op Rb where op is some operation we must define.

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
 op1 = op1 >> 1;
}
```

We can start by mixing different lines into 1. The final line is ```op1 = op1 >> 1;``` and the first line in the while loop is ```if(op1 & 1)```

If the right shift by one bit causes a 1 to be shifted out, then ```FLAGC``` would become 1. This would indicate that the previous value of ```op1 & 1``` was true. If we put the bit shift in first and then use the ```FLAGC``` to determine whether to impliment ```sum = sum + op2_shifted;``` we can skip a few lines of machine code and therefore cycles in the multiplication.

The way we would do this is with a new instruction ```MOVC1 Ra Rb``` which would complete ```Ra := Ra + Rb``` only if ```FLAGC``` was high. 

- [x] Come up with new instruction
- [ ] Impliment Hardware
- [ ] Create new assembly code
- [ ] Test the new software and hardware together

---

### Creating the hardware:

In Issie this would be quite easy to impliment. 

In the ALU sheet, there would be an extra multiplexer to select all the different values of n (1-7) of of ```MOVCn```. These would be connected up to the other 8 option multiplexer which outputs the right output based on the base 8 ALU instructions ```(MOV, ADD, SUB etc...)``` via a 2 option multiplexer with a select based on whether we were doing the normal ```MOV``` or a ```MOVCn```. The select for the new 8 option multiplexer would be the value of n, which is Rc in this case. The inputs would be 0 for 0 and 2-7 as we have nothing defined there yet but input 1 would have the correct value of either ```Ra + Rb``` or just ```Ra```. This would itself be selected by a 2 option multiplexer with a select of ```FLAGC```

Implementing this does require additional inputs into the ALU however as we need to be able to use the register values at address Ra and Rb and also see the "address" of Rc. Currently we only see the register values of Ra and Rb so we will add another input RCADD to the ALU. 

The select for the initial 7 input multiplexer can be implimented by doing the instruction ```!(ALUOPC==0)!(RCADD==0)``` basically checking that the usual outputs would be used if we are in a ```MOV``` instruction but not ```MOVCn``` or if we have any of the other instructions e.g. ```ADD, SUB```.

It all looks like this in Issie (ALU block):

<p align="center">
  <img src="https://github.com/user-attachments/assets/1b5d02e5-7a19-401a-9531-bbbf1d9c9053" width="600"/>
</p>
