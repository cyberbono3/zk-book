# SNARK classification 


In this post I explain SNARK classification. I assume, you are familiar with the basics of zero knowledge proofs. If not, I recommend to take a look at [Entry Guide to Zero knowledge](https://hackmd.io/@aiv768/rJxic3OWle) before reading this post.

# SNARK 
“SNARK” stands for Succinct Non-interactive Argument of Knowledge. 
* succinct means short
* non-interactive means a prover yields a proof. Verifer either accept it or reject.

General purpose SNARKS allow to prove statements about computer programs. 

SNARKS consists of two components:
* front-end. It compiles a statement into arithmetic circuit.
* back-end. It yields a short proof that will be accepted or rejected by a verifier

| Components         | Job                                                                                       | Typical tech             |
| ------------- | ----------------------------------------------------------------------------------------- | ------------------------ |
| **Front‑end** | Compile code + inputs → **arithmetic circuit**  | DSLs: [Circom](https://iden3.io/circom), [Noir](https://aztec.network/noir)             |
| **Back‑end**  | Produce/verify the succinct proof for that circuit                                        | Groth16, Plonk, STARK, … |


---



# Arithmetic circuit
An arithmetic circuit over a finite field is a directed acyclic graph whose gates perform field additions or multiplications. Its leaves are public inputs (verifier-known) and private inputs (witness) for the prover.
Evaluating the circuit yields outputs representing a polynomial function, enabling zero-knowledge proofs by proving knowledge of private inputs without revealing them.

Arithmetic circuit consists of
* one multiplication gate a * b = d
* one addition gate (d + c = out)


![corrected arithmetic circuit ](https://hackmd.io/_uploads/HyMm5FOzeg.png)


---



# Polynomials
Polynomials translate computation checks into polynomial identities. Polynomials fall into few categories:
| Concept                         | Quick definition                                                                         | Why it matters (add links)                                 |
| ------------------------------- | ---------------------------------------------------------------------------------------- | ----------------------------------------------- |
| **Univariate**                  | One variable: p(x) = 5x² + 3x + 2                                                        | Simpler, used by KZG, FRI                       |
| **Multivariate**                | ≥2 variables: f(x,y) = 5x²y³ + 3xy + 2                                                   | Needed for [sumcheck](https://hackmd.io/TkX578g2RTSdv95vtzgOIg), [GKR](https://hackmd.io/reBheKasRWOMyBRsBnup3Q)                        |
| **Multilinear**                 | Each term ≤ 1 deg per var: g(x,y)=5xy+7x+3y                                              | Great for prover‑friendly PIOPs                 |
| **Coefficient form**            | Store the coeffs: \[2, 3, 5] for 5x² + 3x + 2                                            | Smallest to commit, hard to evaluate everywhere |
| **Evaluation form**             | $(p(x_1),p(x_2)....p(x_n))$                                                   |  [Reed Solomon codes](https://dev.risczero.com/reference-docs/about-rs-codes)

---


#  modern SNARK
Modern SNARK consists of two components:
1. Polynomial Interactive Oracle Proof(PIOP)
2. Polynomial Commitment Scheme(PCS)

PIOP + PCS = SNARK
Good practice:
* univariate PIOP + univariate PCS is ok
* multilinear PIOP + multilinear PCS is ok

Not functional:
* ~~univarite PIOP + multilinear PCS~~ is not ok (leads to additional overhead)
* ~~multilinear PIOP + univariate PCS~~ is not ok (leads to additional overhead)


---

## Polynomial Interactive Oracle Proof (**PIOP**) 
Interactive protocol when a prover sends messages to a verifier and some of these messages specify polynomials       


## Polynomial commitment scheme (PCS)
A prover would like to send a polynomial to a verifier. However, this polynomial may be a quite big (degree >= 1000). This introduces a significant сommunication overhead. To mitigate this issue, polynomial commitment scheme(PCS) comes in. 
    PCS gives the verifier confidence, that the prover can not change his mind about what polynomial it has sent. Commitment to a big polynomial is quite short. Essentially, it is either a hash or a field element.
    
General workflow of PCS:
1. The prover sends a commitment C to a polynomial $p$  to the verifier
2. The verifier does not know $p$ and responds with a random challenge $r \in F^n$
3. The verifier asks the prover to open $p$ at this point $r$
4. Finally, the verifier makes sure that the prover indeed knows the polynomial.
![output-2](https://hackmd.io/_uploads/BJiPl5Ozxl.png=50%x)

$p(x) = a_0 + a_1*x + ... + a_{1024}*x^{1024}$
$Com(p) = C$, where p is polynomial, C is commitment





---


# Main PIOP families
1. Quotienting-based (univariate polynomials )
    * **Constant‑round** (1 – 2 rounds ): [Groth16](https://www.rareskills.io/post/groth16), [Plonk](https://vitalik.eth.limo/general/2019/09/22/plonk.html), [Marlin](https://eprint.iacr.org/2019/1047), [STARKs](https://aszepieniec.github.io/stark-anatomy/)

2. Sumcheck-based (multilinear polynomials)
    * **Multi Prover Interactive**: [Spartan](https://eprint.iacr.org/2019/550), [Jolt](https://github.com/a16z/jolt)
    * **IP based**: [GKR](https://hackmd.io/reBheKasRWOMyBRsBnup3Q)   – applies sumcheck layer‑by‑layer


--

## Quotienting (vanishing polynomial)


![corrected arithmetic circuit ](https://hackmd.io/_uploads/HyMm5FOzeg.png)

Quotienting is the well known technique in univariate PIOPs. It bundles all of your circuit’s arithmetic into a single low-degree polynomial identity. 
*   Let consider a multiplicative gate $d = a * b$.
*   Let $g$ be a univariate polynomial, $g(d) = g(a)*g(b)$. 
*   A prover ends $g$ to a verifier 
*   Let $h(d) = g(d)- g(a)*g(b)$ = 0 be a constraint polynomial
*   $d \in S$, $S$ is an interpolation set defined over $n^{th}$ root of unity
**$n^{th}$ root of unity**: $z^n = 1$ (mod p),
Example:  $28^{32} = 1$ (mod 97), 28 is 32nd root of unity 
    
**Fact: if h(d) = 0 exists a quotient polynomial $q$ that is divisible by a vanishing polynomial Z**
If the prover is honest, this equality holds $h(d) = q(d)* Z_s(d)$
$Z_s(d)$ is a vanishing polynomial, $Z_S(x) \;=\; \prod_{d \in S} (x - d)$


* The prover sends $q$ to the verifier
* Verifier picks random $r \in F$ and checks $h(r) = q(r)$, 
  ### if $h(r) = q(r)$ => h = q with high probability (Schwarz Zippel Lemma)



# PCS classification
| PCS family                        | Representative schemes                              | Main pros                                              | Main cons                                                       |
| --------------------------------- | --------------------------------------------------- | ------------------------------------------------------ | --------------------------------------------------------------- |
| **Pairing‑based (cryptographic)** | KZG, HyperKZG (univariate); Zeromorph (multilinear) | Tiny proofs, fast verification                         | Requires trusted setup (SRS); pairing operations (multiplication on elliptic curves); slower prover |
| **Group‑based**                   | Bulletproofs, Hyrax, Ligero, Dory                   | No trusted setup; no pairings needed                   | Verifier time grows with polynomial size (≊ linear)             |
| **Hash‑based**                    | FRI, Brakedown, STIR, Basefold                      | Post‑quantum secure; works over small or binary fields | Large proofs (≈ 100 KB+); higher bandwidth                      |
| **Lattice‑based**                 | Ajtai commitment                                    | Post‑quantum secure                                    | Complex math; bulky proofs/keys                                 |
| **Groups of unknown order**       | DARK                                                | Elegant, no pairings/trusted setup                     | Currently impractical (slow, large parameters)  




---


# Trusted setup (SRS)
Multiparty computation protocol (MPC) is a protocol that  computes a function from multiple private inputs. For instance, a group of people would like to compute a function $f(x_1, x_2,...x_n)$, where every party holds a private input $x_1, x_2,...x_n$.
Trusted setup is MPC aka SRS (structured reference string)
* Let $τ$ be a randomly sampled from a finite field $F$
* $τ$ is a toxic waste, destroyed after protocol completes
* $g$ is generator of a cyclic group
* $D$ is largest degree of polynomials committed



## SRS ($g$, $g^{τ}$, $g^{τ^2}$,..., $g^{τ^D}$) 



---


# KZG Polynomial commitment scheme
## Introduction
* The KZG (or Kate) is most popular pairing-based polynomial commitment scheme.
* Goal: It lets a prover succinctly commit to any unvariate polynomial e.g. $p(x) = 2X^2 + 3x = 2$ by producing a short commitment (an elliptic-curve point of pairing friendly group [BN254](https://hackmd.io/@jpw/bn254) or [BLS12-381](https://hackmd.io/@benjaminion/bls12-381). 
* Once the commitment is published, the prover can not change his mind. Later this commitment can be opened at any point.
* There is a multilinear analog of KZG called [Zeromorph](https://eprint.iacr.org/2023/917.pdf). 
* Requires trusted setup (SRS) 

KZG consists of 3 phases:


## 1. Commit
Univariate polynomial p(x) = $\sum_{i=0}^{n} c_i x^i$
$Com(p)$ =<C,SRS> is a product of coefficient vector and SRS(trusted setup), where $Com(p)$ is commitment to p 
$Com(p)$ = $\prod_{i=0}^{n} {(g^{\tau^i}})^{c_i}$ is called **multiscaler multiplication(MSM)**
In fact, $p(\tau)$ = $\sum_{i=0}^{n}c_i \tau^i$
$Com(p)$ = $g^{p(\tau)}$ = C  (1.0)


The commitment $C$ is a single group element.

---


## 2. Open
A verifier picks $r$ at random and asks a prover to open commitment at $r$. 
The prover claims the opening equals $t$
**Fact: $p(r) = t$  holds if and only if $p(x) - t$ is divisible by $x - r$**
Compute quotient polynomial $w(x)$ of degree n-1
$w(x)= (p(x)- t) / (x - r )$ (1.1)
$Com(w)$ = $g^{w(\tau)}$ (according to 1.0)


## 3. Verify
$(x-r) * w(x) = p(x)$ (1.2) ( according to 1.1)
Verifier checks if this equality 1.2 holds at random point $\tau$ that noone knows
$(\tau-r) * w(\tau) = p(\tau)$ (1.3)
Note: the verifier does not know $p(\tau)$ in 1.3, it knows only $g^{p(\tau)}$
$g^{(\tau-r)} * g^{w(\tau)} = g^{p(\tau)}$ (1.4)
To prove that 1.4 holds, [pairings](https://en.wikipedia.org/wiki/Pairing-based_cryptography) come in


$e(g^{(\tau-r)}, g^{w(\tau)}) = e(g^{p(\tau)}, g)$ (1.5)
if 1.5 holds, 1.4 and 1.3 indeed hold.

**Quick review on pairings**
$g^a, g^b...g^c$ are group elements
Since [DL](https://en.wikipedia.org/wiki/Discrete_logarithm) is hard, a verifier does not know $a,b,c$
Verifier wants to check if $a*b =c$
Pairings allows him to do so 
$e(g^a, g^b) = e(g^c, g)$ (1.5)


---


# Hashing based PCSs

| Scheme          | Polynomial type(s)         | Coding trick / field                                                                  | Key pros                                                  | Key cons / notes                            |
| --------------- | -------------------------- | ------------------------------------------------------------------------------------- | --------------------------------------------------------- | ------------------------------------------- |
| **FRI**         | Univariate                 | Reed–Solomon code over FFT‑friendly field with **blow‑up 2×**; Merkle‑hash commitment | Simple, fast prover, post‑quantum; verifier ≈ O(λ log² n) | Proofs ≈ 100 KB; needs FFT‑friendly field   |
| **STIR / WHIR** | Univariate                 | Tweaks of FRI (different folding / hashing schedules)                                 | Slightly smaller proofs or faster folding                 | Same basic trade‑offs as FRI                |
| **Binius**      | Univariate (binary field)  | Binary‑field analogue of FRI                                                          | Works in tiny 𝔽₂ᵏ;                         | Proofs still large; niche                   |
| **Brakedown**   | Uni‑ & multilinear         | *Any* error‑correcting code; no FFT needed                                            | Faster prover; flexible fields                            | Even larger proofs than FRI → less adoption |
| **Ligero**      | Uni‑ & multilinear         | Reed–Solomon; can view as Brakedown instance                                          | Oldest transparent hash PCS; no trusted setup             | Proofs large; linear‑time verifier          |
| **Basefold**    | Multilinear                | FFT‑friendly folding (multilinear FRI)                                                | Prover‑friendly, matches multilinear PIOPs                | Needs FFT field; proofs still sizable       |
| **FRI‑Binius**  | Multilinear (binary field) | Basefold ported to binary field                                                               | Same small‑field benefits as Binius                       | Proof size ≥ 100 KB; experimental           |

Below I will introduce the most popular hashing based PCS - FRI.

<div style="page-break-after: always; break-after: page;"></div>


---

<!-- .slide: style="font-size: 20px; style="text-align: left" -->
# FRI (Fast Reed-Solomon Interactive Oracle Proofs of Proximity)
FRI 
* heavily used in ZKVMs
* contains log n rounds

It consists of 3 phases:
* Commit
* Fold
* Query


## Commit phase
1. Reed solomon(RS) encoding 
let $p$ = $(p(x_0)...p(x_n))$ be a polynomial , where $x_0...x_n \in F^n$
$F^n$ is FFT friendly field e.g. Goldilocks
RS codeword adds redundancy  $(p(x_0)..,p(x_n),p(x_{n+1})..p(x_{2n}))$
where $x_0...x_{2n} \in F^n$
So, n => 2n,  2 called a blowup factor

2. Merkle hashing of Reed Solomon codeword $H = keccak256(p(x_0)...p(x_{2n}))$


---


## Fold phase
Say $p$ is commited polynomial of degree n
Verifier V picks $r \in F^n$

## Fact: $p(x) = p_{even}(x^2) + r*p_{odd}(x^2)$ is RLP of $p_{even}$ and $p_{odd}$
RLP is random linear combination
$p_{even}$ is a polynomial with event terms
$p_{odd}$ is a polynomial with odd terms


Example: $p(x) = 1 + 2x + 3x^2 + 4*x^3$ is a polynomial p
  $p_{even}(x) = 1 +3x$ are even coefficients of p
  $p_{odd}(x) = 2 +4x$ are odd coefficients of p
  $p_2$ = $p_{even}(x^2) + r*p_{odd}(x^2)$ = $(1 + 3x^2) + r*(2 + 4x^2)$
* ### round 1 
  V picks $r_1 \in F$
 $p_2$ = $p_{even} + r_1*p_{odd}$
 ## $len(p_2) = len(p_1) / 2$
 ## $deg(p_2) = deg(p) / 2$
 P sends $hash(p_2)$ to V
 ...
* ### log n final round 
   $p_{logn}$ is length 2 vector $(t_{final}, t_{final})$
   if prover is honest, $t_{final}$ has degree 0, two entries are equal to each other.
   
## Query phase
Query phase gives V confidence prover P did folding correctly.
In a nutshell, V picks random 128 field elements and checks that P did folding correctly
$p_2 = fold(p_1)$
$p_3 = fold(p_2)$
...
$p_{log_n} = fold(p_{log_{n-1}})$


---


# FRI Batching


Let $w_0$ be $(p(x_0)...p(x_{2_n}))$ of degree $n$
Let $w_1$ be $(p(x_0)...p(x_{n}))$ of degree n/2
 
### To run FRI on $w_0$ and $w_1$
 *  batch them 
   $w_{batched} = w_0 + r_1*w_1$
 * run FRI on result $w_{batched}$

 
### Run FRI on vectors $w_0...w_n$
1.  Affine batching (choose several random parameters $r_1..r_n \in F$)
$w_{batched} = w_0 + r_1*w_1 + r_2*w_2 +...+r_n*w_n$
2.  Parametric batching ( use powers of one random parameter $r \in F$)
$w_{batched} =  w_0 + r^1*w_1 + r^2*w_2 +...+r^n*w_n$















