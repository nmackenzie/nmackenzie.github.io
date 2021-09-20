---
layout: post
title: Intro to Quantum Computing
date: 2021-08-01 14:00:00 +0700
description: Introduction to Quantum Computing. # Add post description (optional)
img: bricks.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Blog]
---
Quantum computing is a novel form of computing which may revolutionize our ability to perform certain types of calculation which are beyond the reach of traditional classical computers. Several companies are actively developing hardware with the goal of natively performing quantum algorithms. There is significant research in developing novel quantum computing algorithms for areas such as cryptography, quantum simulation and machine learning. 

This is an introductory post focused on providing the mathematical background to understand quantum computing to the extent needed to understand the teleportation algorithm, the easiest significant algorithm in quantum computing. The post differs from most introductions in that it makes no use of matrices. Instead, the post introduces quantum computing by focusing on the vector characteristics of the qubits which are at the core of quantum computing.

There are many books on quantum computing, including:

* [Quantum Computation and Quantum Information - Nielsen & Chuang](https://www.amazon.com/Quantum-Computation-Information-10th-Anniversary/dp/1107002176/ref=pd_bxgy_img_1/137-2770317-0425506?pd_rd_w=9ouwl&pf_rd_p=c64372fa-c41c-422e-990d-9e034f73989b&pf_rd_r=XHS5B41F1HPKN67EE0GA&pd_rd_r=d7ff3767-c49b-4e20-b09b-b19aedc1b6e9&pd_rd_wg=TYIi5&pd_rd_i=1107002176&psc=1)

* [Quantum Computer Science - Mermin](https://www.amazon.com/Quantum-Computer-Science-David-Mermin/dp/0521876583)

Microsoft developed the Q# programming language which can be used to implement quantum algorithms. The documentation for Q# includes an [overview](https://docs.microsoft.com/en-us/azure/quantum/overview-qdk) of quantum computing. Microsoft has also created a [Quantum Katas](https://github.com/microsoft/QuantumKatas#introduction) environment for quantum computing and Q#.

# Bits and Vectors
The bit is the primitive unit of information for the digital computer. A single bit has the values 0 or 1 and can represent an entity with two states such as: false (0) or true (1); off (0) or on (1); up or down, etc. These states are mutually exclusive since it is not possible for a bit to have the values 0 and 1 simultaneously. Multiple bits can be associated with each other in a sequence and again be used to represent information. For example, the eight bits 11011000 can represent entities such as the letter Q or the integer 216. From humble beginnings as a hole punched in a card the bit has become central to our digital experience. 

It is natural to consider the consequences of relaxing the constraint that a bit can take on only the values 0 or 1. Vector algebra provides a mathematical technique for this, and we can consider the effect of extending the 0 and 1 values into two vectors defined to have the states \|0> and \|1>. While it is possible to define the addition of two bits in various ways, the result is always either 0 or 1. However, vectors support the concept of superposition in that it is possible to add two vectors together to create a new vector.

We are going to consider a mathematical system where a general vector has the form: 

    |v> = α|0> + β|1> 

where, somewhat unusually, α and β are complex numbers. This system can formally be described as a two-dimensional vector space over the complex numbers. \|0> and \|1> are the basis vectors of this vector space – that is any vector in this vector space can be represented as a linear combination of the two vectors \|0> and \|1> with α and β representing the amount of each basis vector included in the total. Each vector has a length, with the basis vectors \|0> and \|1> defined to be of length 1 and the general vector having a length, L, defined by 
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>L² = α<sup>∗</sup>α + β<sup>∗</sup>β
</code></pre></div></div>

where α* and β* are the complex conjugates of α and β respectively. 

Note that the 0 and 1 in \|0> and \|1> are labels with no other meaning. They could be replaced by \|u> and \|d>, for example, and our analysis would proceed unchanged. However, using \|0> and \|1> allows the simple extension of the classical bit as well as, in more advanced cases, mathematical tricks relying on the labels being the numbers 0 and 1. 

# Functions of Vectors
We can define functions on this vector space. For example, we can define two complex-valued, linear functions, F and G, as follows: 

```
F(|0>) = 0

F(|1>) = 1  

G(|0>) = 1

G(|1>) = 0 
```

Where 0 and 1 are numbers, so that: 

```
F( α|0> + β|1> ) = β 

G( α|0> + β|1> ) = α 
```

The functions F and G measure how similar a vector is to \|1> and \|0> respectively. 

Paul Dirac, who introduced the \|> notation for vectors, developed a concise notation for this type of function. Replacing F by <1\| and G by <0\| the above definitions for F and G become:

```
<1|0> = 0

<1|1> = 1 

<0|0> = 1

<0|1> = 0 
```

So that: 

```
<1|(α|0> + β|1>) = β 

<0|(α|0> + β|1>) = α 
```

<0\| and <1\| are to be understood as linear functions operating on the vectors \|0> and \|1> producing the real numbers 0 or 1 as output. It turns out that the linear functions can also be treated as vectors. Dirac suggested the name bra for vectors like <0\| and the name ket for vectors like \|1> with the names coming from <bra\|ket>. Over 40 pages of his magisterial textbook, [The Principles of Quantum Mechanics](https://www.amazon.com/Principles-Quantum-Mechanics-P-Dirac/dp/1607965607/ref=sr_1_1?dchild=1&qid=1628037103&refinements=p_27%3AP.+A.+M.+Dirac&s=books&sr=1-1&text=P.+A.+M.+Dirac), Dirac showed the expressive power of the bra-ket notation which has much broader applicability than needed here. 

For the general ket, \|v>, defined as: 

    |v> = α|0> + β|1>  

we can associate a bra, <v\|, defined as follows: 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code><v| = α*<0| + β*<1| 
</code></pre></div></div>

The motivation for using the complex conjugate when creating the bra is so we get a real number, not a complex number, when we apply a bra to its source ket, as in: 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>α|0> ->  α*<0| 
</code></pre></div></div>

so that 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>α*α<0|0> = α*α 
</code></pre></div></div>

For a general ket, v, this makes <v\|v> equal to its squared length. 

This type of linear function is referred to as an inner product. It is an extension of the more familiar scalar or dot product. 

We can also consider functions which map a ket into another ket. Using the fact that <0\|0> and <1\|1> are both equal to 1 we can rewrite the kets \|0> and \|1> as follows: 

```
|0> = |0><1|1> 

|1> = |1><0|0> 
```

We can then interpret \|0><1\| as a function which maps \|1> into \|0> and similarly \|1><0\| as a function which maps \|0> into \|1>. These functions are referred to as outer products. 

We can also combine them as follows to get, for example: 

    X = |0><1| + |1><0| 

The way to interpret this is to process the expression from right to left, so applying it to a \|1> leads to: 

```
X|1> = (|0><1| + |1><0|)|1> 

= |0><1|1> + |1><0|1> 

= |0>1 + |1>0 

= |0>
```

and

```
X|0> = (|0><1| + |1><0|)|0> 

= |0><1|0> + |1><0|0> 

= |0>0 + |1>1 

= |1> 
```

This results in: 

```
X|0> = |1>  

X|1> = |0> 
```

The function X is therefore a ket equivalent to the familiar NOT operator for bits. In this context, functions are often referred to as operators – and the two terms are synonymous. 

An inner product, <v\|w>, is a linear function which maps a ket into a complex number. An outer product, \|v><w\|, is a linear function which maps a ket into another ket.

# Pauli Operators 

Outer product notation can be used to define a set of related linear operators, named after Wolfgang Pauli who first described them: 

```
I = |0><0| + |1><1| 

X = |0><1| + |1><0| 

Y = i|1><0| - i|0><1| 

Z = |0><0| - |1><1| 
```

I is the identity operator. The Pauli operators are important in both quantum computing and quantum physics. These can be applied sequentially, so that XY\|v> represents the actions of applying Y to \|v> and then applying X to the resulting ket. 

It is a simple exercise to demonstrate the following relations among the Pauli operators: 

```
1 = X² = Y² = Z² 

iX = YZ = -ZY 

iY = ZX = -XZ 

iZ = XY = -YX 
```
This shows that the Pauli operators do not commute – that is the order in which they are applied is significant.

# Hadamard Operator 

The Hadamard operator is defined as follows: 

```
H|0> = (|0> + |1>) / √2  

H|1> = (|0> - |1>) / √2 
```

This operator converts a pure ket into a superposition of \|0> and \|1> kets. The result is scaled by √2 so that the resulting ket has length 1. 

The Hadamard operator can be defined in terms of Pauli operators and outer products as follows: 

```
H = (X + Z) / √2 

H = (|0><0| + |1><0| + |0><1| - |1><1|) / √2
```

# Qubit 

There is nothing inherently quantum in the description above of a vector space over the complex numbers and the functions operating on that space. However, that mathematics is at the core of quantum computing. Now is the time to add the features which will change the discussion from vector algebra to quantum computing. 

In quantum computing the vectors, \|0> and \|1>, are referred to as qubits (quantum bits.) As above, the general qubit is a vector of the form: 

    |q> = α|0> + β|1>  

Where α and β are complex numbers. This means all the mathematics described above applies unchanged to qubits. A qubit is constrained to be of length 1, so that: 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>1 = α*α + β*β
</code></pre></div></div>

# Observation  

In classical computing when we observe a bit the result is always either 0 or 1. What do we observe when we measure the state of the general qubit? If the qubit is in the state \|0> or \|1> it seems reasonable to assume we would observe it to be in that pure state when we measure it. If the qubit is in the state i\|0> or i\|1> it similarly seems reasonable to assume it would also be in that pure state when we measure it, although doubt remains about the significance of the i since we cannot observe imaginary or complex numbers. 

Assumption. When we measure the state of the general qubit, α\|0> + β\|1>, we will observe it to be either \|0> or \|1> with probability α\*α or β\*β respectively. This observational probability is the core assumption of quantum computing. This assumption is consistent with the reasonableness criteria and length constraints.

# Entanglement 

As with classical bits it is possible to combine two or more bits in a sequence. For example, the general form for two qubits is: 

    |q₁q₀> = α|00> + β|01> + γ|10> + δ|11> 

Where α, β, γ, and δ are complex numbers constrained by: 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>1 = α*α + β*β + γ*γ + δ*δ 
</code></pre></div></div>

\|00>, \|01>, \|10>, and \|11> form a basis for the vector space of two qubits – that is any 2-qubit state is a linear superposition of these basis states. 

We can take two individual qubits and combine them as follows: 

```
|q₁q₀>  = (α₁|0> + β₁|1>)(α₀|0> + β₀|1>) 

        = α₁α₀|00> + β₁α₀|10> + α₁β₀|01> + β₁β₀|11> 
```

This appears to have the same form as the general qubit. However, it transpires there are general two qubit states which cannot be created by taking the product of two individual qubits like this. For example:  

    |q₁q₀> = α|00> + β|11>

with

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>1 = α*α + β*β
</code></pre></div></div>

This qubit cannot be created from the product of two individual qubits and the individual qubits in \|q₁q₀> are deemed to be entangled. The motivation for this is that when we observe the state of this vector we will get the result \|00> with probability α\*α and the result \|11> with probability β\*β. Knowing the observed value of one qubit in the pair, say \|0> we immediately know the value of the other qubit, also \|0> in this example. This means that if we are somehow able to separate the two qubits in the entangled pair we immediately know the value of the second qubit on measuring the first qubit.

# Operations on Multiple Qubits 

We can perform operations on multiple qubits. For example, designating the rightmost qubit in a pair as 0 and the leftmost as 1 (so that Z₁ operates on the leftmost qubit) we can write operations such as: 

```
X₁Z₀|00> = |10> 

X₀Z₁|11> = -|01> 
```

We can also introduce operations on multiple qubits. There is a class of two-qubit operators in which the leftmost qubit controls whether an operation is applied to the rightmost qubit. A controlled-NOT (C<sub>NOT</sub>) applies a NOT operator (X) to the rightmost qubit when the left qubit is \|1> and otherwise makes no change. The C<sub>NOT</sub> is defined as follows: 
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>C<sub>NOT</sub>|00> = |00> 

C<sub>NOT</sub>|01> = |01> 

C<sub>NOT</sub>|10> = |11> 

C<sub>NOT</sub>|11> = |10>
</code></pre></div></div>

Note that similar control can be applied to the other Pauli operators so that a controlled-Z operator (C<sub>z</sub>) is defined by:
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>C<sub>z</sub>|00> = |00> 

C<sub>z</sub>|01> = |01> 

C<sub>z</sub>|10> = |10> 

C<sub>z</sub>|11> = -|11> 
</code></pre></div></div>

C<sub>NOT</sub> and C<sub>z</sub> can be defined as outer products:

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>C<sub>NOT</sub> = |00><00| + |01><01| + |10><11| + |11><10|

C<sub>z</sub> = |00><00| +|01><01| + |10><10| - |11><11|
</code></pre></div></div>


More complex relationships can be derived, such as: 
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>H₀C<sub>NOT</sub> = C<sub>Z</sub>H₀</code></pre></div></div>

# Bell Basis 

It is possible to create a basis from entangled qubits as follows: 

```
|Φ⁺> = (|00> + |11>) / √2 

|Φ⁻> = (|00> - |11>) / √2 

|Ψ⁺> = (|01> + |10>) / √2 

|Ψ⁻> = (|01> - |10>) / √2 
```

This is known as the Bell basis. A combination of C<sub>NOT</sub> and Hadamard operators can be used to transform between the computational basis and the Bell basis as follows: 

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>C<sub>NOT</sub>H₁|00> = (|00> + |11>) / √2 

C<sub>NOT</sub>H₁|11> = (|00> - |11>) / √2 

C<sub>NOT</sub>H₁|01> = (|01> + |10>) / √2 

C<sub>NOT</sub>H₁|10> = (|01> - |10>) / √2 
</code></pre></div></div>

Note that this process is reversible so that, for example: 
<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>H₁C<sub>NOT</sub> = (|00> + |11>) / √2 = |00>
</code></pre></div></div>

This reverse process is referred to as "measuring in the Bell basis."

We can extend this process from two qubits to an arbitrary number of qubits with the general n-dimension qubit having 2ⁿ terms like α\|011...10> - one for each combination of values of each individual qubit. This exponential growth in complexity makes it impossible to simulate many qubits on a classical computer because it lacks the memory required to store the state of an arbitrary n-dimensional qubit. This is one of the drivers behind the development of quantum computers which use novel techniques to store and manipulate qubits – and so would allow quantum computing algorithms requiring large numbers of qubits to be implemented. 

# Teleportation 

Teleportation is a simple algorithm using qubits to create an interesting result – the transfer of quantum state from one place to another. 

The goal of teleportation is for Alice to transfer a quantum state to Bob. The no-cloning theorem states that it is not possible to clone a quantum state, so it is not possible for Alice to simply clone a qubit and send it to Bob. 

The general idea of the teleportation algorithm is that Alice and Bob each possess one qubit of a pair of entangled qubits in one of the Bell states. Alice has another qubit the state of which she wants to transfer or "teleport" to Bob. She does this by performing some operations on the two qubits she has and observing the state of both her qubits. Depending on what she observes, Alice then tells Bob to perform specific operations on his qubit which will transform his qubit into precisely the state as the initial qubit Alice had. 

Alice's initial qubit is in general form. 

    |v> = α|0> + β|1> 

Alice and Bob share an entangled pair of qubits, where Alice has the leftmost and Bob the rightmost: 

    |Φ⁺> = (|00> + |11>) / √2 

The complete initial state of the system is therefore: 

    (α|0> + β|1>)(|00> + |11>) / √2 

Alice performs a C<sub>NOT</sub> operation on her qubits (the two leftmost qubits) leaving the system as follows: 

    α|0>(|00> + |11>) / √2 + β|1> (|10> + |01>) / √2 

Alice performs a Hadamard operation on the leftmost qubit: 

    (α|0> + α|1>)(|00> + |11>) / 2 + (β|0> - β|1>) (|10> + |01>) / 2 

Which gives: 

```
α (|000> + |100> + |011> + |111>) / 2 

+ β (|010> -|110> + |001> - |101>) / 2 
```
This can be regrouped on the two leftmost qubits as follows: 

```
|00> (α|0> + β|1>) / 2 + 

|10> (α|0> - β|1>) / 2 + 

|01> (α|1> + β|0>) / 2 + 

|11> (α|1> - β|0>) / 2 
```

When Alice measures her two qubits, the observation selects one of the four groups. If Alice’s qubits are \|00> then Bob already has precisely the qubit Alice wanted to transfer. If Alice has any of the other groups, she sends Bob specific operations he must perform on his qubit to transform it into the desired form. 

Alice has | Bob does operation 
--|--
\|00> | 1 
\|10> | Z 
\|01> | X 
\|11> | ZX 

After performing the operation and observing the qubit, Bob's qubit will be in the desired state - the one that Alice started with. Of course, when Bob measures that qubit he will find it to be in the state \|0> or \|1> with probabilities α\*α or β\*β respectively.

# Challenge of Quantum Computing 

Quantum computing has two core challenges: the development of useful algorithms and the development of novel hardware which allows the implementation of quantum computing algorithms.  

Quantum computing is not intended to replace traditional general-purpose computing whether that be on a phone or in a datacenter. Algorithm development is focused on specialized needs beyond the current capability of general-purpose computing. Hardware development is focused on the creation of physical systems where qubits, with the mathematical properties described earlier, can be created and controlled thereby allowing the implementation of these algorithms. 

# Summary 

Quantum computing is based on three concepts: superposition, entanglement and observational probability. Superposition and entanglement are consequences of use of vectors to describe states. The quantum nature arises because of the use of probability to provide a physical meaning when the state of qubits is observed 