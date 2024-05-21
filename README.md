# Tate Pairing and Miller Loop

In identity-based cryptosystems, the computational efficiency of bilinear pairings is crucial to the overall performance of the cryptographic algorithm. Currently, all fast calculation methods for bilinear pairings are built upon the Miller's algorithm. Current research on fast computation of bilinear pairings focuses on how to reduce the number of loops in Miller's algorithm to improve computational efficiency.
The main purpose of this document is to analyze the principles of the commonly used Tate Pairing and optimization methods for the Miller Loop.
## 1. Preliminaries
### 1.1 Pairing
In cryptography, a Pairing refers to a mapping from two additive groups to a multiplicative group:
$$e:G_1×G_2 \rightarrow G_T$$
Where $e$ is a function, $G_1$ and $G_2$ are additive groups, $G_T$ is a multiplicative group, and this mapping satisfies:Bilinearity,Non-degeneracy,Computability.

The key property utilized here is bilinearity, which means the function has two input variables and satisfies linear mapping for both individually. In other words, if we fix either input variable, the resulting unary function exhibits linear mapping, expressed as:
$$e(P,Q+R)=e(P,Q) \cdot e(P,R)$$
$$e(P+S,Q)=e(P,Q) \cdot e(S,Q)$$
$$e(aP,bQ)=e(P,Q)^{ab}$$
### 1.2 The Order of Elliptic Curve Point Groups and Torsion Points
The order of a point group on an elliptic curve is denoted as $\#E(F_q)$, representing the total number of points on the curve, including the point at infinity.
Let $m \geq 1$ be an integer. A point $P$ on an elliptic curve is said to have order $m$ if $mP = \mathcal{O}$, where $\mathcal{O}$ is the point at infinity. The set of all points of order $m$ forms a subgroup called the $m$-torsion group (or torsion subgroup), also referred to as points of order $m$ or $m$-torsion points.
$$ E[m] \triangleq \{ P \in E:mP=\mathcal{O} \}$$ 
### 1.3 Embedding Degree
There is a curve $E(F_q)$. Let $r$ be a large enough prime factor and $r$ divides $q^k-1$. However, for $0<i<k$, $r\nmid p^i-1$, we call $k$ the embedding degree with respect to $r$.
### 1.4 Divisor
Divisors of an elliptic curve are defined as
$$ [E]= \triangleq \sum_{P \in E} n_P[P]  $$
Essentially, divisor is a set of points. For example, if we have $E(F_{61}):y^2=x^3+8x+1$, and points $ P=(57,24)$, $Q=(25,37)$ lie on this curve, we can define some divisors as $D_1=1[P]+2[Q] $ , $ D_2=2[P]+3[Q] $ etc., $ [E]$ is a free ***abel group*** generated by $ E(F_{q}) $.

The degree of a divisor is the sum of the coefficients in the aforementioned expression.
$$ deg([E])=  \sum_{P \in E} n_P  $$
The sum of divisor, denoted as $ sum([E])=  \sum_{P \in E} n_PP  $, is still a point on the elliptic curve.

$ [E] $ is called a Divisor of a rational function on $ E $ if and only if both $ deg([E])=0 $ and $ sum([E])=\mathcal{O} $ hold simultaneously.
### 1.5 Pairing Function
Given functions $f$ and $g$, if $[f] = \sum_{P \in E} n_P [P]$ and $[g]= \sum_{P \in E} m_P [P]$ have different supports (support refers to those elliptic curve points where the divisor coefficient is not 0), then
$$ f([g])=g([f])$$
This serves as the foundation for pairing computation. The key to constructing pairing functions is to find a practically computable $ f_{r,P}$ that satisfies the condition.
$$ [f_{r,P}]=r[P]-r[\mathcal{O}]   $$    
Where $ P \in E(r)$.

The specific calculation method involves defining a family of functions, whose divisor is given by:
$$ [f_{m,P}]=m[P]-[mP]-(m-1)[\mathcal{O}]   $$  
Note that when $ m = r $, since $ rP = \mathcal{O} $, the divisor aligns with $ [f_{r,P}]=r[P]-r[\mathcal{O}]   $. Subsequently, we can compute $f_{m+1,P}$  through the following recursive relation
$$ f_{m+1,P}=f_{m,P} \cdot  \frac{l_{mP,P}}{v_{(m+1)P}} $$  
In this equation, $ l_{mP,P} $ represents the function corresponding to the line passing through points $ mP $ and $ P $, $ v_{(m+1)P} $ represents the function corresponding to the vertical line passing through point $ (m + 1)P $ and intersecting the $ x $ -axis.
## 2. Tate Pairing
The Tate Pairing was introduced by mathematician John Tate and later extended by cryptographer Stephen Lichtenbaum. It is sometimes referred to as the Tate-Lichtenbaum Pairing. In cryptography, the Tate Pairing finds wider application due to its faster computation compared to the Weil Pairing.

Let $E(F_q)$ be an elliptic curve defined over a finite field $F_q$, where $q$ is a large prime number. Let $r$ be a large prime such that $r \mid \#E(F_q)$ (i.e., $r$ divides the order of the elliptic curve) and $r$ is coprime to $q$. Let $k$ be the embedding degree of $E(F_q)$ with respect to $r$. Choose $P \in E(F_{q^k})[r]$, where $E(F_{q^k})[r]$ is the $r$-torsion group of $E(F_{q^k})$, denoted simply as $E[r]$. We have $ \#E(r)=r^2$, and $r^2 \mid E(F_{q^k})$.

For a prime number $r$, $E[r]$ can be divided into $r + 1$ subgroups of order $r$. One of these subgroups consists entirely of points on $E(F_q)$, and this subgroup is unique.

Let $r$ be coprime to $q$. Assume we have a function whose divisor is
 $$div(f_{r,P}) = r[P]  -[rP]-(r-1)[\mathcal{O}]$$
 The Tate pairing is defined as the non-degenerate bilinear pairing: $ t(P,Q)=\langle P, Q \rangle_r = f_{r,P}(Q+R)/f_{r,P}(R)$,where $R$ is a randomly chosen point on $E(F_{q^k})$, $ t $ is a relationship: 
$$ t: E(F_{q^k})[r] \times \frac{E(F_{q^k})}{rE(F_{q^k})} \rightarrow \frac{F_{q^k}^*}{(F_{q^k}^*)^r}$$


Since the result of the pairing is an element of the quotient group, to ensure a unique value for the Tate pairing, an exponentiation by $(q^k - 1)/r$ is required.
The standardized Tate pairing is defined as:
$$
\hat{t}(P, Q) = \langle P, Q \rangle_r^{(q^k - 1)/r},
$$
where $P$ and $Q$ are points on the elliptic curve, and $\hat{t}(P, Q)$ is defined and non-zero as long as both $P$ and $Q$ are defined and non-zero.
## 3. Miller's Algorithm
The core of Miller's algorithm is to construct a rational function $f_{r,P}$ on an elliptic curve whose divisor satisfies the Miller equation: $div(f_{r,P}) = r[P]  -[rP]-(r-1)[\mathcal{O}]$. The algorithm has a complexity of $O(\log r)$. A commonly used implementation of the Miller loop is as follows.

![The Miller Loop](<fig/Miller loop.png>)

*Fig. 1 The Miller Loop*

The parameters involved are:

$r$: The prime factor, which is a parameter of the elliptic curve, represented in binary form as $r = (r_{n-1}...r_1r_0)_2$, where each bit is either 0 or 1.

$P$ and $Q$: Points in the $r$-torsion group of the elliptic curve.

$f_{r,P}(Q)$: The value of the pairing function $f_{r,P}$ at the point $Q$.

$l_{T, P}$: The line passing through points $T$ and $P$ in the plane of the elliptic curve.

$l_{T, P}(Q)$: The value of $l_{T, P}$ at point $Q$.

$v_{T+P}$: The vertical line passing through the point resulting from the addition of $T$ and $P$.

$v_{T+P}(Q)$: The value of $v_{T+P}$ at point $Q$.

The algorithm works by initializing $T$ and $f$ and then iteratively calculating $T$ and $f$ based on whether the binary digits of $r$ are 1 or 0.
## 4. Improving the efficiency of Miller's Algorithm 
Enhancing the efficiency of Miller's algorithm can be achieved through two primary approaches: 
improving the construction of the bilinear pairing itself or finding optimal parameters for Miller's algorithm under specific conditions. 

Below are some common optimization techniques for Miller's algorithm that leverage specific parameters:

***(1) Final Exponentiation Optimization: Optimizing the final exponentiation step in the pairing computation can significantly reduce the overall complexity.***

***(2) Precomputation: In bilinear pairing computations, one of the input points is often a fixed base point. Precomputing values related to scalar multiplication and line computations involving this base point can reduce the complexity of point multiplication.***

***(3) Optimization of Parameter Selection in Miller's Algorithm: Choosing optimal parameters for the algorithm, such as the curve and the finite field, can lead to efficiency gains.***

***(4) Optimization of Loop Complexity in Miller's Algorithm: Selecting a low Hamming weight $r$ (i.e., a representation of $r$ with fewer non-zero bits) can reduce the number of iterations in the Miller loop, thereby improving efficiency.***

For example, in [2], the optimized algorithm is written as:

![](<fig/Optimized  Miller loop.png>)

*Fig. 2 The Optimized Algorithm of Miller Loop*

The optimizations here are: 

$ \blacktriangle $ Using a low Hamming weight $r$ to reduce the number of loops (represented as $e=\sum_{i=0}^{L-1}s_i2^i for s_i \in \{ -1,0,1\}$ in the algorithm), 

$ \blacktriangle $ Precomputing lines ($l_{\lambda,\mu}$)

$ \blacktriangle $ Simplifying the final exponentiation.
## 5. Implementation of Miller's Algorithm
### 5.1 Miller's Algorithm in Groth16
In the Groth16 code, the Rust implementation of the Miller Loop relies on the external library Pairing 0.23.0 [3]. The source file lib.rs defines an interface applicable to pairing-friendly curves. According to the library documentation, the concrete implementation of this interface is based on the BLS12-381 curve. The source code for this implementation depends on the bls12_381 library [4, 5].

For the implementation of a normal Miller's Algorithm, there are:
```rust
fn miller_loop<D: MillerLoopDriver>(driver: &mut D) -> D::Output {
    let mut f = D::one();

    let mut found_one = false;
    for i in (0..64).rev().map(|b| (((BLS_X >> 1) >> b) & 1) == 1) {
        if !found_one {
            found_one = i;
            continue;
        }

        f = driver.doubling_step(f);

        if i {
            f = driver.addition_step(f);
        }

        f = D::square_output(f);
    }

    f = driver.doubling_step(f);

    if BLS_X_IS_NEGATIVE {
        f = D::conjugate(f);
    }

    f
}

``` 

* Rust Code: The Rust Implementation of Miller's Algorithm 

```rust
impl MillerLoopDriver for Adder {
        type Output = Fp12;

        fn doubling_step(&mut self, f: Self::Output) -> Self::Output {
            let coeffs = doubling_step(&mut self.cur);
            ell(f, &coeffs, &self.p)
        }
        fn addition_step(&mut self, f: Self::Output) -> Self::Output {
            let coeffs = addition_step(&mut self.cur, &self.base);
            ell(f, &coeffs, &self.p)
        }
        fn square_output(f: Self::Output) -> Self::Output {
            f.square()
        }
        fn conjugate(f: Self::Output) -> Self::Output {
            f.conjugate()
        }
        fn one() -> Self::Output {
            Fp12::one()
        }
    }

``` 

* Rust Code: Definitions for Driver-related Operations

The Rust code aligns with the algorithm flowchart illustrated in Figure 1, featuring two key functions:

$ \blacktriangle $ Function ***driver.doubling_step(f)***

*This step encompasses two operations: point doubling on the elliptic curve and line evaluation(Corresponding to steps 3 and 4 in Fig. 1; or steps 6 and 7 in Fig. 2).*

$ \blacktriangle $ Function ***driver.addition_step(f)***

*This step also includes two operations: point addition on the elliptic curve and line evaluation(Corresponding to steps 6 and 7 in Fig. 1; or steps 12 and 13 in Fig. 2).*

### 5.2 An Enhanced Miller's Algorithm

In Section 4, we mentioned four directions for Miller loop optimization, as follows:

*  *Final Exponentiation Optimization*

* *Precomputation*

*  *Optimization of Parameter Selection in Miller's Algorithm*

*  *Optimization of Loop Complexity in Miller's Algorithm*

Reference [2] presents an enhanced Miller's algorithm, incorporating optimizations such as reducing the number of iterations, precomputing Lines and  eliminating the final exponentiation. The figure below illustrates a comparative analysis of its operational characteristics against the conventional algorithm.

![alt text](<fig/Algorithm Compare.png>)

*Fig. 3 Comparison of Algorithms Before and After Optimization. ① Replace binary encoding with NAF encoding to reduce the Hamming weight of the encoding; ② Optimize line evaluation using Precomputing Lines.*

In Fig.3, we can note that the new algorithm mainly involves two optimization processes, which are: 
* Replace binary encoding with NAF encoding to reduce the Hamming weight of the encoding;  
* Optimize line evaluation using Precomputing Lines.

The third optimization is a replacement operation: replace the final exponentiation with a series of progressively less expensive, equivalent checks.

### 5.3 Multi Miller Loop

For the computation of Multi Miller Loop, the Rust code is as follows.
```rust
pub fn multi_miller_loop(terms: &[(&G1Affine, &G2Prepared)]) -> MillerLoopResult {
    struct Adder<'a, 'b, 'c> {
        terms: &'c [(&'a G1Affine, &'b G2Prepared)],
        index: usize,
    }

    impl<'a, 'b, 'c> MillerLoopDriver for Adder<'a, 'b, 'c> {
        type Output = Fp12;

        fn doubling_step(&mut self, mut f: Self::Output) -> Self::Output {
            let index = self.index;
            for term in self.terms {
                let either_identity = term.0.is_identity() | term.1.infinity;

                let new_f = ell(f, &term.1.coeffs[index], term.0);
                f = Fp12::conditional_select(&new_f, &f, either_identity);
            }
            self.index += 1;

            f
        }
        fn addition_step(&mut self, mut f: Self::Output) -> Self::Output {
            let index = self.index;
            for term in self.terms {
                let either_identity = term.0.is_identity() | term.1.infinity;

                let new_f = ell(f, &term.1.coeffs[index], term.0);
                f = Fp12::conditional_select(&new_f, &f, either_identity);
            }
            self.index += 1;

            f
        }
        fn square_output(f: Self::Output) -> Self::Output {
            f.square()
        }
        fn conjugate(f: Self::Output) -> Self::Output {
            f.conjugate()
        }
        fn one() -> Self::Output {
            Fp12::one()
        }
    }

    let mut adder = Adder { terms, index: 0 };

    let tmp = miller_loop(&mut adder);

    MillerLoopResult(tmp)
}
```

Multi Miller Loop computation distinguishes itself from single-point computation primarily in its approach to point doubling and point addition. For single-point calculations, Algorithm 1 is implemented sequentially, following its steps. However, multi-point operations utilize a preprocessed G2Prepared structure as input, enabling parallel point doubling and point addition across all points.

References：

[1]	Miller V. The Weil pairing and its efficient calculation. Journal of Cryptology, 2004, 17: 235-261.

[2]	https://eprint.iacr.org/2024/640.pdf

[3] https://github.com/zkcrypto/pairing

[4] https://github.com/zkcrypto/bls12_381

[5] https://docs.rs/bls12_381/latest/bls12_381
