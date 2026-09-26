## Logic and proofs
| Notation                        | Meaning                               |
| ------------------------------- | ------------------------------------- |
| $\top$, `true`                  | True                                  |
| $\bot$, `false`                 | False                                 |
| $\neg P$, $\lnot P$, `!P`       | Not $P$                               |
| $P\land Q$, `P && Q`            | $P$ and $Q$                           |
| $P\lor Q$, `P \| Q`             | $P$ or $Q$                            |
| $P\oplus Q$, $P\veebar Q$       | Exclusive or                          |
| $P\Rightarrow Q$, $P\to Q$      | $P$ implies $Q$                       |
| $P\Leftarrow Q$                 | $P$ is implied by $Q$                 |
| $P\Leftrightarrow Q$, $P\iff Q$ | $P$ if and only if $Q$                |
| $P\equiv Q$                     | Logical equivalence                   |
| $\forall x\,P(x)$               | For every $x$, $P(x)$                 |
| $\exists x\,P(x)$               | There exists an $x$ satisfying $P(x)$ |
| $\exists!x\,P(x)$               | There exists exactly one such $x$     |
| $\nexists x\,P(x)$              | No such $x$ exists                    |
| $\therefore$                    | Therefore                             |
| $\because$                      | Because                               |
| $P\vdash Q$                     | $Q$ is formally derivable from $P$    |
| $P\models Q$                    | $P$ semantically entails $Q$          |
| $\models P$                     | $P$ is valid in every interpretation  |
| $\blacksquare$, $\square$, QED  | End of proof                          |
| $:=$, $\coloneqq$               | Defined as                            |
| $\stackrel{\text{def}}{=}$      | Equal by definition                   |
## Basic set notation
|Notation|Meaning|
|---|---|
|$A,B,S,X$|Sets|
|$\{a,b,c\}$|Set containing $a,b,c$|
|$\{\}$, $\varnothing$, $\emptyset$|Empty set|
|$\{x\mid P(x)\}$|Set of $x$ such that $P(x)$|
|$\{x:P(x)\}$|Alternative set-builder notation|
|$x\in A$|$x$ is an element of $A$|
|$x\notin A$|$x$ is not an element of $A$|
|$A\ni x$|$A$ contains $x$|
|$\lvert A\rvert$|Cardinality (number of elements) of $A$|
|$A=B$|Sets contain exactly the same elements|
|$A\ne B$|Sets are unequal|
|$A\subseteq B$|$A$ is a subset of $B$|
|$A\subsetneq B$, $A\varsubsetneqq B$|$A$ is a proper subset of $B$|
|$A\supseteq B$|$A$ is a superset of $B$|
|$A\supsetneq B$|$A$ is a proper superset of $B$|
|$A\nsubseteq B$|$A$ is not a subset of $B$|
|$\mathcal P(A)$, $2^A$|Power set: the set of all subsets of $A$|
|$\{A_i\}_{i\in I}$|Family of sets indexed by $I$|

## Set operations
|Notation|Meaning|
|---|---|
|$A\cup B$|Union|
|$A\cap B$|Intersection|
|$A\setminus B$, $A-B$|Set difference|
|$A^c$, $\overline A$, $U\setminus A$|Complement of $A$|
|$A\triangle B$, $A\mathbin{\Delta}B$|Symmetric difference|
|$\bigcup_{i\in I}A_i$|Union of an indexed family|
|$\bigcap_{i\in I}A_i$|Intersection of an indexed family|
|$\bigcup A$|Union of all sets belonging to $A$|
|$\bigcap A$|Intersection of all sets belonging to $A$|
|$A\uplus B$, $A\sqcup B$|Disjoint union|
|$A\mathbin{\dot\cup}B$|Union known to be disjoint|
|$A\perp B$|Sometimes: $A$ and $B$ are disjoint|
|$A\cap B=\varnothing$|$A$ and $B$ are disjoint|

## Products, tuples, sequences, and intervals
| Notation                          | Meaning                                         |
| --------------------------------- | ----------------------------------------------- |
| $(a,b)$                         | Ordered pair                                    |
| $(a_1,\ldots,a_n)$              | Ordered $n$-tuple                             |
| $A\times B$                     | Cartesian product                               |
| $A^n$                           | $n$-fold Cartesian product                    |
| $\prod_{i\in I}A_i$             | Cartesian product of an indexed family          |
| $(a_n)_{n\in\mathbb N}$         | Sequence                                        |
| $\langle a_1,\ldots,a_n\rangle$ | Tuple or sequence, especially in CS             |
| $A^*$                           | All finite strings/sequences over $A$         |
| $A^+$                           | All nonempty finite strings over $A$          |
| $A^\omega$                      | Infinite sequences over $A$                   |
| $\varepsilon$, $\lambda$      | Empty string, depending on convention           |
| $[a,b]$                         | Closed interval                                 |
| $(a,b)$                         | Open interval                                   |
| $[a,b)$, $(a,b]$              | Half-open intervals                             |
| $]a,b[$                         | Alternative notation for $(a,b)$              |
| $[n]$                           | Often $\{1,\ldots,n\}$, but convention varies |
## Common number sets
| Notation                                  | Meaning                                                          |
| ----------------------------------------- | ---------------------------------------------------------------- |
| $\mathbb N$                             | Natural numbers; either $\{0,1,\ldots\}$ or $\{1,2,\ldots\}$ |
| $\mathbb N_0$                           | Natural numbers including zero                                   |
| $\mathbb Z$                             | Integers                                                         |
| $\mathbb Z^+$, $\mathbb Z_{>0}$       | Positive integers                                                |
| $\mathbb Z_n$, $\mathbb Z/n\mathbb Z$ | Integers modulo $n$                                            |
| $\mathbb Q$                             | Rational numbers                                                 |
| $\mathbb R$                             | Real numbers                                                     |
| $\mathbb R^n$                           | Real $n$-dimensional space                                     |
| $\mathbb C$                             | Complex numbers                                                  |
| $\mathbb F$, $\mathbb K$              | An arbitrary field                                               |
| $\mathbb F_q$, $\operatorname{GF}(q)$ | Finite field with $q$ elements                                 |
| $\{0,1\}^n$                             | Bit strings of length $n$                                      |
| $\{0,1\}^*$                             | All finite binary strings                                        |
## Relations

A binary relation from $A$ to $B$ is a subset

$$
R\subseteq A\times B.
$$

|Notation|Meaning|
|---|---|
|$aRb$|$a$ is related to $b$|
|$(a,b)\in R$|Explicit equivalent of $aRb$|
|$a\not Rb$|$a$ is not related to $b$|
|$R:A\leftrightarrow B$|Relation between $A$ and $B$|
|$\operatorname{dom}(R)$|Domain of $R$|
|$\operatorname{ran}(R)$, $\operatorname{im}(R)$|Range/image of $R$|
|$R^{-1}$, $R^\smallsmile$|Converse relation|
|$R\circ S$|Composition of relations|
|$R^n$|$n$-fold relational composition|
|$R^*$|Reflexive-transitive closure|
|$R^+$|Transitive closure|
|$R?$|Often reflexive closure in regular-expression notation|
|$R\restriction_A$, $R\vert_A$|Restriction of $R$ to $A$|
|$\operatorname{Id}_A$, $\Delta_A$|Identity relation on $A$|
|$A/R$|Quotient set of equivalence classes|
|$[a]_R$|Equivalence class of $a$|
|$a\sim b$|$a$ is equivalent or related to $b$|
|$a\cong b$|$a$ and $b$ are structurally isomorphic|
|$a\simeq b$|Equivalent up to a specified notion|
|$a\approx b$|Approximately or contextually equivalent|
### Properties of binary relations

| Property             | Formal condition                         |
| -------------------- | ---------------------------------------- |
| Reflexive            | $\forall x\in A,\;xRx$                 |
| Irreflexive          | $\forall x\in A,\;\neg(xRx)$           |
| Symmetric            | $xRy\Rightarrow yRx$                   |
| Antisymmetric        | $xRy\land yRx\Rightarrow x=y$          |
| Asymmetric           | $xRy\Rightarrow\neg(yRx)$              |
| Transitive           | $xRy\land yRz\Rightarrow xRz$          |
| Total/connected      | $xRy\lor yRx$ for distinct $x,y$     |
| Equivalence relation | Reflexive, symmetric, and transitive     |
| Preorder             | Reflexive and transitive                 |
| Partial order        | Reflexive, antisymmetric, and transitive |
| Total order          | Partial order plus totality              |
| Strict order         | Usually irreflexive and transitive       |
## Order notation

|Notation|Meaning|
|---|---|
|$a<b$, $a>b$|Strict order|
|$a\le b$, $a\ge b$|Non-strict order|
|$a\preceq b$, $a\succeq b$|Abstract order|
|$a\prec b$, $a\succ b$|Abstract strict order|
|$a\parallel b$|$a$ and $b$ are incomparable|
|$a\lessdot b$, $a\prec\!\cdot\, b$|$b$ covers $a$|
|$\min A$, $\max A$|Minimum and maximum|
|$\inf A$, $\sup A$|Infimum and supremum|
|$\bot$, $\top$|Least and greatest elements|
|$a\wedge b$|Meet|
|$a\vee b$|Join|
|$(P,\le)$|Ordered set|
|$\operatorname{Min}(A)$|Set of minimal elements|
|$\operatorname{Max}(A)$|Set of maximal elements|
## Functions and mappings

A function is commonly formalized as a relation in which every input has exactly one output.

|Notation|Meaning|
|---|---|
|$f:A\to B$|Function from $A$ to $B$|
|$x\mapsto f(x)$|Mapping rule|
|$f:x\mapsto f(x)$|Defines $f$ by its action|
|$f(a)$|Value of $f$ at $a$|
|$f:A\rightharpoonup B$|Partial function|
|$f:A\hookrightarrow B$|Injective map|
|$f:A\twoheadrightarrow B$|Surjective map|
|$f:A\xrightarrow{\sim}B$|Isomorphism or bijection|
|$B^A$, $A\to B$|Set/type of all functions $A\to B$|
|$\operatorname{dom}(f)$|Domain|
|$\operatorname{codom}(f)$|Codomain|
|$\operatorname{im}(f)$, $f(A)$|Image/range|
|$f[S]$, $f(S)$|Direct image of $S\subseteq A$|
|$f^{-1}[T]$, $f^{-1}(T)$|Preimage of $T\subseteq B$|
|$f^{-1}$|Inverse function, if it exists|
|$g\circ f$|Composition: first $f$, then $g$|
|$\operatorname{id}_A$, $1_A$|Identity function on $A$|
|$f\vert_S$, $f\restriction_S$|Restriction of $f$ to $S$|
|$f^n$|$n$-fold composition, or power depending on context|
|$f^{(n)}$|$n$th derivative or iterate, context-dependent|
|$\ker f$|Kernel|
|$\operatorname{graph}(f)$|Set $\{(x,f(x)):x\in A\}$|
|$\operatorname{supp}(f)$|Support of $f$|
|$\operatorname{Fix}(f)$|Fixed points $\{x:f(x)=x\}$|
### Function properties

|Term|Condition|
|---|---|
|Injective / one-to-one|$f(x)=f(y)\Rightarrow x=y$|
|Surjective / onto|$\forall y\in B,\exists x\in A:f(x)=y$|
|Bijective|Both injective and surjective|
|Endofunction|$f:A\to A$|
|Identity|$\operatorname{id}_A(x)=x$|
|Constant|$f(x)=c$ for every $x$|
|Involution|$f\circ f=\operatorname{id}$|
|Idempotent|$f\circ f=f$|
## Anonymous, higher-order, and lambda functions

|Notation|Meaning|
|---|---|
|$\lambda x.\,E$|Anonymous function with body $E$|
|$\lambda x:E$|Alternate lambda notation|
|$x\mapsto E$|Ordinary mathematical equivalent|
|$\lambda x\,y.\,E$|Abbreviation for $\lambda x.(\lambda y.E)$|
|$(\lambda x.E)\,a$|Apply the function to $a$|
|$E[x:=a]$|Substitute $a$ for free occurrences of $x$|
|$\alpha$-equivalence|Equality up to renaming bound variables|
|$\to_\beta$|Beta reduction|
|$\to_\eta$|Eta reduction|
|$f\circ g$|Function composition|
|$f^n$|Repeated application/composition|
|$\operatorname{curry}(f)$|Convert a multiargument function to nested functions|
|$\operatorname{fix}(f)$, $Yf$|Fixed-point operation|
Examples:

$$ \lambda x.\,x+1 \qquad x\mapsto x+1 \qquad f(x)=x+1 $$

all express closely related descriptions of the successor function.

## Families and indexed operations

|Notation|Meaning|
|---|---|
|$(a_i)_{i\in I}$|Indexed family|
|$\{a_i:i\in I\}$|Set of values in the family|
|$\sum_{i\in I}a_i$|Indexed sum|
|$\prod_{i\in I}a_i$|Indexed product|
|$\bigcup_{i\in I}A_i$|Indexed union|
|$\bigcap_{i\in I}A_i$|Indexed intersection|
|$\bigsqcup_{i\in I}A_i$|Indexed disjoint union|
|$\coprod_{i\in I}A_i$|Coproduct/disjoint union|
## Cardinality and infinite sets

|Notation|Meaning|
|---|---|
|$\lvert A\rvert$|Cardinality (number of elements) of $A$|
|$\#A$|Alternative notation for the cardinality of $A$|
|$A\sim B$, $A\approx B$|Sometimes equinumerous|
|$A\preceq B$|Sometimes cardinal domination|
|$\aleph_0$|Cardinality of $\mathbb N$|
|$\aleph_\alpha$|Infinite aleph cardinal|
|$\mathfrak c$|Cardinality of the continuum|
|$2^{\aleph_0}$|Cardinality of $\mathcal P(\mathbb N)$|
|$\kappa^+$|Successor cardinal of $\kappa$|
|$\operatorname{cf}(\kappa)$|Cofinality of $\kappa$|
|$\omega$|First infinite ordinal|
|$\omega_1$|First uncountable ordinal|

## Computer-science notation

| Notation                  | Meaning                                        |
| ------------------------- | ---------------------------------------------- |
| $x\leftarrow v$, `x := v` | Assignment                                     |
| $x=v$, `x == v`           | Mathematical equality / programming comparison |
| $x\ne v$, `x != v`        | Inequality                                     |
| `f(x)`                    | Function or procedure call                     |
| `x -> e`, `λx.e`          | Lambda function                                |
| $e:T$, $e\in T$           | Expression $e$ has type $T$                    |
| $\Gamma\vdash e:T$        | Under context $\Gamma$, $e$ has type $T$       |
| $A\times B$               | Product type                                   |
| $A+B$, $A\sqcup B$        | Sum/variant type                               |
| $A\to B$                  | Function type                                  |
| $A^*$                     | List/string type or Kleene closure             |
| $A?$, `Option<A>`         | Optional value                                 |
| $\mu X.F(X)$              | Recursive/least fixed-point type               |
| $\nu X.F(X)$              | Greatest fixed-point type                      |
| $e\Downarrow v$           | Expression evaluates to value $v$              |
| $e\to e'$                 | One evaluation/reduction step                  |
| $e\to^*e'$                | Zero or more evaluation steps                  |
| $[\![e]\!]$               | Meaning/denotation of $e$                      |
| $L(M)$                    | Language recognized by machine $M$             |
| $\Sigma$                  | Alphabet                                       |
| $\Sigma^*$                | All finite strings over $\Sigma$               |
| $\varepsilon$             | Empty string                                   |
| $uv$, $u\cdot v$          | String concatenation                           |
| $R\mid S$, $R+S$          | Regex choice/union                             |
| $R^*$                     | Kleene star                                    |
| $R^+$                     | One or more repetitions                        |
| $\delta(q,a)$             | State-transition function                      |
| $\mathcal O(f(n))$        | Asymptotic upper bound                         |
| $\Omega(f(n))$            | Asymptotic lower bound                         |
| $\Theta(f(n))$            | Tight asymptotic bound                         |
| $o(f(n))$, $\omega(f(n))$ | Strict asymptotic bounds                       |

## Database relation notation

|Notation|Meaning|
|---|---|
|$R(A_1,\ldots,A_n)$|Relation/schema with attributes|
|$\sigma_P(R)$|Select rows satisfying $P$|
|$\pi_A(R)$|Project onto attributes $A$|
|$R\bowtie S$|Natural join|
|$R\Join_P S$|Conditional/theta join|
|$R\times S$|Cartesian product|
|$R\cup S$, $R\cap S$, $R-S$|Set operations on compatible relations|
|$\rho_{S}(R)$|Rename relation or attributes|
|$R\ltimes S$|Semijoin|
|$R\div S$|Relational division|
