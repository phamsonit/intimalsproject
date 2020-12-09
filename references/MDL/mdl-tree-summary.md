* Original Data *

We distinguish two types of non-terminals: those having lists of children (in 
which repetition of non-terminals may occur), and those having fixed sets of 
children (some of which may be optional).

- lists:
each non-terminal in the list has an independent probability.
The code length of a non-terminal with probability p is -log(p).
Furthermore, we have a probability of p' to stop the list or to continue. 

For instance, if we have two equally likely symbols A and B, as well as 
probability to stop of 0.25, the coding length of list 
ABA is -log(1-0.25)-log(0.5)-log(1-0.25)-log(0.5)-log(1-0.25)-log(0.5)-log(0.25)

- fixed sets of children:
each optional child has a probability of p of being included.
For each optional child we encode whether or not it is included using log(p) bits. 

The code is based on a traversal of the AST: for instance, if we do a depth-
first traversal of the AST, we encode each node in this order.

* Patterns *

The code for a pattern consists of:

* an encoding of the non-terminal in its root,
using a uniform distribution over all non-terminals present in the AST.
I propose to use a uniform distribution to avoid a bias for certain non-terminals

* an encoding for each internal node in the pattern,
 where we distinguish two types of non-terminals:
- lists:  we use the code for children in the data here.
- fixed children:
 we use a code length of 1 bit to encode for each
 of the optional children whether or not that optional child is in the pattern.

Finally, for every child we use 1 bit to encode whether or not that child is 
included as an internal node in the pattern as well.

* an encoding of the number of occurrences of the pattern.
If N is the number of nodes in the data for the non-terminal in the root of the pattern, this 
could be achieved using log(N) bits. 

For instance, for the pattern S(A(CD)B) over grammar:
S -> {A,B}*
A -> C D? (D is optional)
the coding length would be:
- -log(1/5) (5 non-terminals in the grammar; encoding for the root)
- -log(1-0.25) - log(0.5) - log(1-0.25) - log(0.5) - log(0.25) (children of S using codes for lists)
- 2 bit for the 2 children of S to encode the fact that A is internal, B is not
- 1 bit to encode that the optional non-terminal D is present below A
- 2 bit to encode that both children of A are leafs in the pattern
- 16 bit to encode the number of occurrences of the pattern (assuming the data has 2^16 occurrences of the non-terminal S)

If we would use more/less than 1 bit, this would reflect a different type of 
prior over how large patterns should be, and which non-terminal we want in the 
root of a pattern.

* Data with one pattern *

Assume that S is the root of the pattern, and has M occurrences among the N 
nodes for S in the AST. Then for every node labeled S in the AST, we will use 
-log(M/N) bit if the pattern occurs, or -log(1-M/N) bit if the pattern does 
not occur. (khó à nghen)

If the pattern does not occur, the original code is used subsequently to 
encode the children.

If the pattern occurs, I propose to encode the children as follows:
- first we encode how many children there are, similar to the case described 
above: for a probability of stopping of p', for n children, we use a code of 
length - ( n - k ) * log(1-p) - log(p), where k is the number of children in 
the pattern. (The argument is that due to the presence of the pattern, we 
already know that there is a certain number of children; we only need to 
encode how many more there are.)
- then we need to encode which of the children belong to the pattern. 
I've reflected on this, but I think this is a better solution. 
Essentially among the n children we would want to distribute the k children in 
the pattern. I think it would be preferable if all of the possibilities would 
have the same probability, that is the probability of each possibility should 
be 1/(n over k). I propose to a code of length log(n over k) to encode which 
children belong to the pattern.
This code is shorter than the one we discussed earlier today.
- finally, for all children that are not in the pattern, we use the normal 
process to encode which non-terminal is in that child. 
- for children that are defined by the pattern, the process for nodes within 
the pattern is repeated: we encode the number of children, which children 
belong to the pattern, ...

I think this code can be extended towards multiple patterns and even to 
overlapping patterns (with some more effort).

Do you follow this description? If not, we can also discuss this further later 
this week, or I could create some further example(s).

A remaining concern that I have is that patterns will need to be quite large 
in order to allow for compression. I will still reflect a little bit further 
on that concern.



* Problem 1:
Input: tree database, set of patterns
Output: subset of patterns that minimise encode database

- How to encode database
- How to encode patterns 


* Problem 2:
Input: tree database
Output: set of patterns that minimise encoding database

- How to encode database
- How to encode patterns 
- How to mini the set of patterns


