# DECA-Y1-Spring-Challenge-1: Multiplication Speed-up
This is the repo for my DECA Y1 Spring Term Challenge work (multiplication speed-up design).

# Version 1 Design Inspirations (Feb 16th)
* This multiplication aims to directly yield the result of multiplication between two 16-bit numbers, `A(15:0)` and `B(15:0)`. As the main multiplication block takes only two 8-bit numbers (due to full adder constraints), `A` and `B` are split into two parts each - `AL = A(7:0)`, `AH = A(15:8)`, `BL = B(7:0)`, and `BH = B(15:8)`. It is easily proved that `A * B = AL * BL + 256(AL * BH + AH * BL) + 65536(AH * BH)`. Hence, if only the **least significant (LS) 16 bits** are desired (suitable to store in a single register), it is proposed that `{A * B}(15:0) = {AL * BL + 256(AL * BH + AH * BL)}(15:0)`, without considering the `AH * BH` part which would purely contribute to the most significant (MS) 16 bits of the final product.

* Note: Version 1 Design utilizes a total of **64 full adders** (which is the limit given in lab handout), and outputs only the **least significant (LS) 16 bits** of the final product to `Ra`.

# Main aspects improved in EEP1
1) Designed the `ADD889` block, comprised of one 8-bit adder which has its `COUT` and `SUM` connected as a 9-bit output value; <br>
   ![image](https://github.com/user-attachments/assets/88735951-0efe-4398-b232-fbec647e0644)
2) Designed an `8*8 multiplication block` which **outputs a 16-bit number**, including seven `ADD889` blocks; <br>
   ![image](https://github.com/user-attachments/assets/d91845a9-7e2f-4e35-aee4-4450a59ca656)
3) Designed an `MLU` (short for `Multiplication Unit`) which takes another 8-bit adder and the multiplication block to perform multiplication; <br>
   ![image](https://github.com/user-attachments/assets/9a37a417-3de3-46a4-80f3-d93c440b1d7d)
4) Used spare bits of INS according to the lab handout (see below);
5) Updated `DPDECODE` block and `DATAPATH` sheet for correct output and new instruction selection, without inteferencing existing ones. <br>
   ![image](https://github.com/user-attachments/assets/4e506c86-cf38-453b-aa79-10d6519e05b7)

* A sample test file is also included, named `multest.txt` (with its corresponding `.dgm` file). <br>

# New instructions included
Before introducing new instructions, some formats are described here: Our inputs are from `Ra(15:0)` and `Rb(15:0)`, with final output `Ra(15:0)` which is the **Least Significant 16 bits** of the multiplied result.

`MOVC1 Ra, Rb` -> `Ra = PPR := Ra(7:0) * Rb(7:0)` <br>
`MOVC2 Ra, Rb` -> `PPR(15:8) := {Ra(7:0) * Rb(15:8)}(7:0) + PPR(15:8)`; `Ra := PPR` <br>
`MOVC3 Ra, Rb` -> `PPR(15:8) := {Ra(15:8) * Rb(7:0)}(7:0) + PPR(15:8)`; `Ra := PPR` <br>

* Note: It is assumed that these three new instructions are used **in this particular sequence** (i.e. `MOVC1` -> `MOVC2` -> `MOVC3`) to generate correct result.

# Example Instructions
`MOV R1, #0x5F` &emsp; // which is 0x005F **sign extended** <br>
`MOV R2, #0xB1` &emsp; // which is 0xFFB1 **sign extended** <br>
`MOVC1 R1, R2` <br>
`MOVC2 R1, R2` <br>
`MOVC3 R1, R2` &emsp;  // expected output is 0xE2AF for the **LS 16 bits** of the product <br>

# Version 2 Design (Feb 17th)
* Based on the same design principle of Version 1 Multiplier, the higher 16 bits of the results are also taken into consideration. It is proposed that `{A * B}(15:0) = {AL * BL + 256(AL * BH + AH * BL) + 65536(AH * BH)}(15:0)`, as well as `{A * B}(31:16) = {AL * BL + 256(AL * BH + AH * BL) + 65536(AH * BH)}(31:16)`.

* In Version 2 design, the **full 32-bit product** could be obtained, and only **56 full-adders** are used.

# Main aspects improved
* The internal structure of the MLU is improved to support more newly-defined instructions, and the output values are more meaningful - which contributes the output of the overall product. <br>
![image](https://github.com/user-attachments/assets/af8960ad-6ed0-47da-b4c1-cee8676ddf62)

<br>

**Main components updated:**
* `REG1` and `REG2`: These two 16-bit registers **locks** the values from `Ra` and `Rb` for multi-cycle processing;
* `MULT8`: It takes two 8-bit numbers and outputs a 16-bit product;
* `Partial Product Register (PPR)`: It is a 32-bit register which stores partial products from `AL * BL`, `AL * BH` and `AH * BL`;
* `MUX1` and `MUX2`: selects the source of input according to `SEL` signal;
* `MUX3` and `MUX4`: selects what number should be written to `PPR` and be the output to `Ra`.

Another huge advancement is that **only 56 full adders** are used in this design (which saves 8 compared to Version 1).

# New instructions included
Note: in the following comments, `RxL` means `Rx(7:0)` and `RxH` means `Rx(15:8)`. <br>
* `MOVC1 Ra, Rb` &emsp;  // `PPR(15:0) := RaL * RbL`; `Ra(15:0) := 0x00(15:8) + {{RaL * RbL}(15:8)}(7:0)` <br>
* `MOVC2 Ra, Rb` &emsp;  // `PPR := PPR`; `Ra(15:0) := RaL * RbH`  <br>
* `MOVC3 Ra, Rb` &emsp;  // `PPR := PPR`; `Ra(15:0) := RaH * RbL` <br>
* `MOVC4 Ra, Rb` &emsp;  // `PPR(15:8) := Ra(15:8)`; `Ra(15:0) := 0x00(15:8) + {Ra(15:8)}(7:0)` -> updates PPR with the newly calculated bits <br>
* `MOVC5 Ra, Rb` &emsp;  // `PPR := PPR`; `Ra(15:0) := RaH * RbH` <br>
* `MOVC6 Ra, Rb` &emsp;  // `PPR := PPR`; `Ra(15:0) := PPR(15:0)` -> get LS 16 bits of the final product <br>

Note that the above instructions should be used **in particular sequence** (i.e. as a **subroutine**), demonstrated below, to achieve the equivalent result of `Ra := {Ra * Rb}(31:16)` and `Rb := {Ra * Rb}(15:0)`: <br>

`0x00` `MOV Ra, #num_1` <br>
`0x01` `MOV Rb, #num_2` <br>
`0x02` `MOVC1 Ra, Rb` &emsp;&ensp;  // `PPR(7:0)` written; `Ra` loaded with `0000 0000 XXXX XXXX` <br>
`0x03` `MOVC2 Rb, Ra` &emsp;&ensp;  // `Rb` loaded with `XXXX XXXX XXXX XXXX` from `MUL` <br>
`0x04` `ADD Ra, Ra, Rb` &emsp;  // sum updated to `Ra` <br>
`0x05` `MOVC3 Rb, Ra` &emsp;&ensp;  // `Rb` loaded with `XXXX XXXX XXXX XXXX` from `MUL` <br>
`0x06` `ADD Ra, Ra, Rb` &emsp;  // sum updated to `Ra`, which is the correct bit sequence of the final product (from the 8th to the 23rd bit) <br>
`0x07` `MOVC4 Ra, Rb` &emsp;&ensp;  // `PPL` has its 8th to 23rd bits updated to `Ra`; meanwhile, `Ra` has its internal bits shifted 8 positions rightwards <br>
`0x08` `MOVC5 Rb, Ra` &emsp;&ensp;   // `Rb` loaded with `XXXX XXXX XXXX XXXX` from `MUL` (which is `AH * BH`) <br>
`0x09` `ADC Ra, Ra, Rb` &emsp;  // `ADC` takes `FlagC` into account; the content of `Ra` is itself the MS part of the final product <br>
`0x0A` `MOVC6 Rb, Ra` &emsp;&ensp;  // `Rb` loaded with the LS part of the final product from `PPL` <br>

Hence, if not considering the two `MOV` instructions at the start, the overall multiplication process would take **9 clock cycles** to execute for **any 16-bit number**, yielding a **32-bit overall result** (which is stored in two separate destination registers) - which is a huge advance compared to the original "while loop" method.

# Example Instructions
`0x00` `EXT 0xFC` <br>
`0x01` `MOV R1, #0xB1` <br>
`0x02` `EXT 0x01` <br>
`0x03` `MOV R2, #0xCB` <br>
`0x04` `MOVC1 R1, R2` <br>
`0x05` `MOVC2 R2, R1` <br>
`0x06` `ADD R1, R1, R2` <br>
`0x07` `MOVC3 R2, R1` <br>
`0x08` `ADD R1, R1, R2` <br>
`0x09` `MOVC4 R1, R2` <br>
`0x0A` `MOVC5 R2, R1` <br> 
`0x0B` `ADC R1, R1, R2` <br>
`0x0C` `MOVC6 R2, R1` <br>

The above instructions calculate the product of `0xFCB1` and `0x01CB` and yield `0x01C5115B` as the final product, stored in `Reg1` and `Reg2`, which is the correct result. <br>

A waveform simulation of the above instructions was done in Issie: <br>
![image](https://github.com/user-attachments/assets/48dd3e42-23c5-4a1b-b2d9-63d60aa51a78)

# Evaluation
* In Version 2 design, the **full 32-bit product** could be obtained, and only **56 full adders** are used.
* It could only carry out **unsigned** numtiplication.
* As the total number of full adders are limited, this multiplication is implemented using a **subroutine**. If more adders could be used, the multiplication process could be done in only one clock cycle (with **significantly increased hardware cost**).
