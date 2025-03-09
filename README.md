# DECA_Section2_Challenge
Challenge work for DECA Spring Term Section 2

---

### Table of Contents

 1. [Background](#background)
 2. [Implimentation](#implimentation)
 3. [Coming up with new instructions](#coming-up-with-new-instructions)
 4. [Creating the hardware](#creating-the-hardware)
 5. [Create new assembly code](#create-new-assembly-code)
 6. [Testing the new software and hardware out](#Testing-the-new-software-and-hardware-out)
 7. [Evaluation](#evaluation)

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

The select for the initial 7 input multiplexer can be implimented by checking that the usual outputs would be used if we are in a ```MOV``` instruction but not ```MOVCn``` or if we have any of the other instructions e.g. ```ADD, SUB```. Checking between ```MOV``` and ```MOVCn``` can be done by first checking if ```ALUOPC=0``` otherwise not in ```MOV``` at all, then checking that ```INS[8]=0``` otherwise we definitely are using ```MOV``` as it is using an ```IMM``` as the value we set the register at ```Ra``` to, and finally checking that ```RCADD = 0``` otherwise those 3 bits are used to define what ```n``` is in ```MOVCn```

It all looks like this in Issie (ALU block):

<p align="center">
  <img src="https://github.com/user-attachments/assets/06c7752b-b689-4fa2-be5d-963c341d7357" width="600"/>
</p>

- [x] Come up with new instruction
- [x] Impliment Hardware
- [ ] Create new assembly code
- [ ] Test the new software and hardware together

---

### Create new assembly code:

The original assembly code looked like this:

```
Setting values for op1 and op2:
MOV R0, #12
MOV R1, #5

Setting initial values for other variables:
MOV R3, #0
MOV R2, R1

While loop condition:
CMP R0, #0
JEQ 9

If statement:
MOV R4, R0
AND R4, #1
CMP R4, #1
JNE 2

Adding op2_shifted to sum:
ADD R3, R3, R2

Shifting op2_shifted and op1:
LSL R2, R2, 1
LSR R0, R0, 1

Jumping back to beginning of while loop:
JMP 65527 (−9)
```

The new code could instead look like this:

```
Setting values for op1 and op2:
MOV R0, #12
MOV R1, #5

Setting initial values for other variables:
MOV R3, #0
MOV R2, R1

While loop condition:
CMP R0, #0
JEQ 5

Shift op1 by 1 bit:
LSR R0, R0, 1

ADD if FLAGC is high:
MOVC1 R3, R2

Shifting op2_shifted and op1:
LSL R2, R2, 1

Jumping back to beginning of while loop:
JMP 65531 (−5)
```

This reduces the assembly down to 10 from 14 instructions and could save 4 cycles each time ```FLAGC``` is high (a 1 in the LSB of sum before the shift) and 3 cycles if ```FLAGC``` is low as we run the ```MOVC1``` instruction no matter what rather than jumping.

- [x] Come up with new instruction
- [x] Impliment Hardware
- [x] Create new assembly code
- [ ] Test the new software and hardware together

---

### Testing the new software and hardware out:

First I will test this hardware out with the original case of __12*5__:

This works perfectly with the correct values in the register files

<p align="center">
  <img src="https://github.com/user-attachments/assets/abe96335-bd30-48cd-9a5d-6c54b96c5d0b" width="600"/>
</p>

- [x] Come up with new instruction
- [x] Impliment Hardware
- [x] Create new assembly code
- [x] Test the new software and hardware together

---

### Evaluation

The new instruction I have added reduces the assembly down to 10 from 14 instructions and could save 4 cycles each time ```FLAGC``` is high (a 1 in the LSB of sum before the shift) and 3 cycles if ```FLAGC``` is low as we run the ```MOVC1``` instruction no matter what rather than jumping.

It also reduces the number of registers we need to use by 1 as R4 isn't needed for the result of the AND instruction.

Some limitations of this are that 3 new multiplexers are needed plus 2 new inputs are needed for the ```ALU``` and 1 new output from the ```DPDECODE``` have been added. This is quite hardware costly and so may not be worth it in some architectures.

Overall this can be used for both signed and unsigned multiplication very effectively and speeds the multiplication up a lot with only the addition of 1 new instruction.

If I was to improve on this, I would work to impliment a full hardware based solution to 16-bit multiplication. This would require the input of lots of adders and temporary registers in order to brute force it however and so would be much more hardware costly. It might not be a good weighting between clock cycle increase and price and quantity of hardware and so a non brute force method might need to be implimented such as Intel x86 “MUL” and “IMUL” Instructions or ARM “MUL” and “MLA” (Multiply‐Accumulate) Instructions
