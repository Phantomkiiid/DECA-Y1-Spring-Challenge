# DECA-Y1-Spring-Challenge
This is the repo for my DECA Y1 Spring Term Challenge work (multiplication speed-up design).

# Design Inspirations
This multiplication aims to directly yield the result of multiplication between two 16-bit numbers, `A(15:0)` and `B(15:0)`. As the main multiplication block takes only two 8-bit numbers (due to full adder constraints), `A` and `B` are split into two parts each - `AL = A(7:0)`, `AH = A(15:8)`, `BL = B(7:0)`, and `BH = B(15:8)`. It is easily proved that `A * B = AL * BL + 256(AL * BH + AH * BL) + 65536(AH * BH)`. Hence, if only the **least significant (LS) 16 bits** are desired (suitable to store in a single register), it is proposed that `{A * B}(15:0) = {AL * BL + 256(AL * BH + AH * BL)}(15:0)`, without considering the `AH * BH` part which would purely contribute to the most significant (MS) 16 bits of the result.

# Main aspects improved in EEP1
1) Designed the `ADD889` block, comprised of one 8-bit adder which has its `COUT` and `SUM` connected as a 9-bit output value;
   ![image](https://github.com/user-attachments/assets/88735951-0efe-4398-b232-fbec647e0644)
2) Designed an `8*8 multiplication block` which **outputs a 16-bit number**, including seven `ADD889` blocks;
   ![image](https://github.com/user-attachments/assets/d91845a9-7e2f-4e35-aee4-4450a59ca656)
3) Designed an `MLU` (short for `Multiplication Unit`) which takes another 8-bit adder and the multiplication block to perform multiplication;
   ![image](https://github.com/user-attachments/assets/9a37a417-3de3-46a4-80f3-d93c440b1d7d)
4) Used spare bits of INS according to the lab handout (see below);
5) Updated `DPDECODE` block and `DATAPATH` sheet for correct output and new instruction selection, without inteferencing exesting ones.
   ![image](https://github.com/user-attachments/assets/4e506c86-cf38-453b-aa79-10d6519e05b7)

A sample test file is also included, named `multest.txt` (with its corresponding `.dgm` file).

Before introducing new instructions, some formats are described here:
Our inputs are from `Ra(15:0)` and `Rb(15:0)`, with final output `Ra(15:0)` which is the Least Significant 16 bits of the multiplied result.

# New instructions included
`MOVC1, Ra, Rb` -> `Ra = PPR := Ra(7:0) * Rb(7:0)` <br>
`MOVC2, Ra, Rb` -> `PPR(15:8) := {Ra(7:0) * Rb(15:8)}(7:0) + PPR(15:8)`; `Ra := PPR` <br>
`MOVC3, Ra, Rb` -> `PPR(15:8) := {Ra(15:8) * Rb(7:0)}(7:0) + PPR(15:8)`; `Ra := PPR` <br>

* Note: It is assumed that these three new instructions are used **in this particular sequence** (i.e. `MOVC1` -> `MOVC2` -> `MOVC3`) to generate correct result.

* Example Usage:

`MOV R1, #0x5F`  // which is 0x005F **sign extended** <br>
`MOV R2, #0xB1`  // which is 0xFFB1 **sign extended** <br>
`MOVC1 R1, R2` <br>
`MOVC2 R1, R2` <br>
`MOVC3 R1, R2`   // expected output is 0xE2AF for the **LS 16 bits** of the product <br>

