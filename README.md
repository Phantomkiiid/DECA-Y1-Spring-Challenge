# DECA-Y1-Spring-Challenge
This is the repo for my DECA Y1 Spring Term Challenge work (multiplication speed-up design).

# Main aspects improved in EEP1
1) Designed the `ADD889` block, comprised of one 8-bit adder which has its `COUT` and `SUM` connected as a 9-bit output value;
   ![image](https://github.com/user-attachments/assets/88735951-0efe-4398-b232-fbec647e0644)
2) Designed a `8*8 multiplication block` which **outputs a 16-bit number**, including seven `ADD889` blocks;
   ![image](https://github.com/user-attachments/assets/d91845a9-7e2f-4e35-aee4-4450a59ca656)
3) Designed a `MLU` (short for `Multiplication Unit`) which takes another 8-bit adder and the multiplication block to perform multiplication;
   ![image](https://github.com/user-attachments/assets/9a37a417-3de3-46a4-80f3-d93c440b1d7d)
4) Used spare bits of INS according to the lab handout (see below);
5) Updated `DPDECODE` block and `DATAPATH` sheet for smooth output and new instruction selection.
   ![image](https://github.com/user-attachments/assets/4e506c86-cf38-453b-aa79-10d6519e05b7)

A sample test file is also included, named `multest.txt` (with its corresponding .dgm file).

Before introducing new instructions, some formats are described here:
Our inputs are from `Ra(16:0)` and `Rb(16:0)`, with final output `Ra(16:0)` which is the Least Significant 16 bits of the multiplied result.

# New instructions included
`MOVC1, Ra, Rb` -> `Ra = PPR := Ra(7:0) * Rb(7:0)` <br>
`MOVC2, Ra, Rb` -> `PPR(15:8) := {Ra(7:0) * Rb(15:8)}(7:0) + PPR(15:8)`; `Ra := PPR` <br>
`MOVC3, Ra, Rb` -> `PPR(15:8) := {Ra(15:8) * Rb(7:0)}(7:0) + PPR(15:8)`; `Ra := PPR` <br>

*Note: It is assumed that these three new instructions are used **in this particular sequence** (i.e. `MOVC1` -> `MOVC2` -> `MOVC3`) to generate correct result.

*Example Usage:

`MOV R1, #0x5F`  // which is 0x005F **sign extended** <br>
`MOV R2, #0xB1`  // which is 0xFFB1 **sign extended** <br>
`MOVC1 R1, R2` <br>
`MOVC2 R1, R2` <br>
`MOVC3 R1, R2`   // expected output is 0xE2AF for the **LS 16 bits** of the product <br>

