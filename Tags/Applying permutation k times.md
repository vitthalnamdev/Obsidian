I have a permutation , p.  p[i] here denotes , that I have to send the arr[i] to p[i] in 1 move.
This operation ,have to be done for every 1<=i<=n.
When , performed this operation K times , when K is very large . Then , what will be the final permutation. 
1. Brute Force: Do the operation of shifting  every element in an array , K times . It has complexity of O(K) . K can be something around 1e18.
2. Binary Exponentiation: Can be done in O(log(K)) , How ? Simple , I try to alter the permutation array p . Initially p has value for one 1 step , right ? Now , I will try to make it 2 step , arr[i] first go to p[i] , then it will go to p[p[i]] , right ? and then p[p[p[i]]] , ... and so on . What if I have the value of K/2 permutation in halfpower array . then , arr[i] will go to p[p[p[p.... k/2 times]]] or p[halfpower[i]] .
   Refer the link: https://cp-algorithms.com/algebra/binary-exp.html
3. Can also do it with permutation graph.
   **Note:** This task can be solved more efficiently in linear time by building the permutation graph and considering each cycle independently. You could then compute   k<math xmlns="http://www.w3.org/1998/Math/MathML"><mi>k</mi></math>$k$  modulo the size of the cycle and find the final position for each number which is part of this cycle.