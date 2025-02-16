# DECA-Y1-Spring-Challenge
This is the repo for my DECA Y1 Spring Term Challenge work (multiplication speed-up design).

Main aspects improved in EEP1:
1) Designed the "ADD889" block, comprised of one 8-bit adder which has its "COUT" and "SUM" connected as a 9-bit output value;
2) Designed a "8*8" multiplication block which outputs a 16-bit number, including seven ADD889 blocks;
3) Designed a "MLU" (short for "Multiplication Unit") which takes another 8-bit adder and the multiplication block to perform multiplication;
4) Used spare bits of INS according to the lab handout.
