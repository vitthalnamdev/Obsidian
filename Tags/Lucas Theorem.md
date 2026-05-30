
The lucas theorem is generally , used to calculate nCr mod p.
When p is smaller prime number.
Because , when not use lucas theorem ,and calculating the nCr with the general way that I know, the denominator will 0 as  the prime is small, which is preventing me to calculate the value of nCr when n is small.

Let p be a prime number, and let r and c be written in p-ary notation.
$$
\begin{array}{l} r = {r_k} \cdots {r_2}{r_1}{r_0} = {r_0} + {r_1}p + {r_2}{p^2} + \cdots + {r_k}{p^k} \qquad (0 \le {r_i} \lt p) \\ c = {c_k} \cdots {c_2}{c_1}{c_0} = {c_0} + {c_1}p + {c_2}{p^2} + \cdots + {c_k}{p^k} \qquad (0 \le {c_i} \lt p) \\ \end{array}
$$

Then,
$$
\left( {\begin{array}{*{20}{c}} r \\ c \\ \end{array}} \right) = \left( {\begin{array}{*{20}{c}} {{r_0}} \\ {{c_0}} \\ \end{array}} \right)\left( {\begin{array}{*{20}{c}} {{r_1}} \\ {{c_1}} \\ \end{array}} \right)\left( {\begin{array}{*{20}{c}} {{r_1}} \\ {{c_1}} \\ \end{array}} \right) \cdots \left( {\begin{array}{*{20}{c}} {{r_k}} \\ {{c_k}} \\ \end{array}} \right)(\bmod p)
$$
