# Stream Cipher

[【完结合集】Dan Boneh的密码学 第一季【伊卡酱全程译注】](https://www.bilibili.com/video/BV1yx411T78i?p=1&vd_source=e729e179d71c2ffd261e2600a9b03b63)

[Cryptography I Stanford University](https://www.coursera.org/learn/crypto/home/week/1)

## Definition of Symmetric Ciphers

A cipher defined over $(K, M, C)$ is a pair of "efficient" algorithms $(E, D)$ where
$$
E:K \times M \longrightarrow C\\
D:K\times C\longrightarrow M
$$
and $\forall m\in M,k\in K:$
$$
s.t.\ D(k,E(k,m))=m\ \ \ (1.1)
$$
The equation (1.1) is usually called consistency requirement of a cipher.

> What is efficient? Different people has different ideas. If you're inclined towards theory, efficient means runs in polynomial time. But if you're more practically inclined, efficient means runs in a certain time period, like within one minute.

E is often randomized, and D is always deterministic.

## Information Theoretic Security

This is something about information theory. The first person to study the security of ciphers rigorously is very very famous Claude Shannon. In 1949, he published a  famous paper *Communication Theory of Secrecy Systems* which proposed an idea of Perfect Security.

The basic idea of Shannon is that if you're just an eavesdropper getting to see a single CT, then it should reveal "no" info about the PT.

**Def:** a cipher $(E,D)$ over $(K,M,C)$​​ has perfect security if

$\forall m_0,m_1\in M,c \in C$ and $len(m_0)=len(m_1):$
$$
Pr[E(k,m_0)=c]=Pr[E(k,m_1)=c]\ \ \ (1.2)
$$
where $k \overset{R}{\leftarrow}K$​.

**Comment:**

1. Given CT can't tell if msg is $m_0$ or $m_1$​​ for all m.
2. Even the most powerful adv. learns nothing about the PT from CT.
3. There's no CT only attack!! But other attacks maybe possible.

## One Time Pad (OTP)

The one time pad was invented by Vernam in 1917. It's the first example of a "secure" cipher.

### Is OTP a cipher?

For OTP, $M=K=C=\{0,1\}^n$ and $|M|=|K|=|C|$​. Actually, the key k is a random bit string as long as our msg. 
$$
c:=E(k,m)=k \oplus m\ \ \ (1.3)
$$

> $\oplus$ is xor which means addition modulo 2. For example, 1 $\oplus$ 1 = 1 + 1 mod 2 $\equiv$ 0 (mod 2). So what the encryption alg does is calculate the addition then modulo 2 bit by bit.

$$
D(k,c)=k\oplus c\ \ \ (1.3)
$$

Indeed, let's prove that the OTP is satisfied with the consistency requirement.

*Proof:*
$$
D(k,E(k,m))=k \oplus E(k,m)=k \oplus k \oplus m = 0 \oplus m = m\ \Box
$$

> That's one of the xor properties: k $\oplus$ k = k + k mod 2 = 2k mod 2 = 0. 

From a performance point of view, OTP is super super fast. Unfortunately, it's difficult to use in practice. The length of its key is too long so that if you're able to transmit your key in a secure way, there's no necessity to use a cipher, right? Just send your plaintext because it's the same length with your key k. :smile:

### Whether it is secure?

Let's focus a little bit on security.

**Lemma:** OTP has perfect secrecy.

*Proof:* 

$\forall m \in M,c \in C$​, if the number of keys s.t. $E(k,m)=c$ is $\lambda$,
$$
Pr_{k \overset{R}{\leftarrow}K}[E(k,m)=c]=\frac{\lambda}{|K|}=const
$$
It's trivial to say that for OTP, num of keys that map m to c is only 1.$\ \ \ \Box$

### The bad news...

Are there other ciphers that has perfect secrecy and possibly have much, much shorter keys?

The bad news is that Shannon has proved it's impossible.

**Thm:** If a cipher has perfect security, then $|K| \geq |M|$​.

## Pseudo Random Generator (PRG)

How to make OTP into a practical scheme, namely using short key to guarantee "security"? The core of stream cipher is its key is not truly-random but pseudo-random.

**Def:** PRG is a func. $G:\{0,1\}^s \longrightarrow\{0,1\}^n,n>>s$ that map from seed space to a n-bit string and is "efficiently" computable by det. alg.

**Comment:**

1. G is definitely deterministic.
2. The output should look random.
3. PRG must be unpredictable.

### Stream Cipher

Let's introduce our leader Stream Cipher:
$$
c=E(k,m):=m \oplus G(k)
$$

$$
D(k,c)=G(k)\oplus c\ \ \ (1.4)
$$

Can a stream cipher have perfect secrecy? No, since teh key must be as long as the message!

So right now, we need a different definition of security and it will depend on specific PRG.

### PRG must be unpredictable

Suppose PRG is predictable,
$$
\exist i:G(k)|_{1,...,i}\overset{alg.}{\rightarrow}G(k)|_{i+1,...,n}
$$
then the stream cipher is insecure. Why?

Just by some prior knowledge, the attacker actually knows the initial part of the message happens tobe some known value. For example, we know in the mail protocol, SMTP, the standard mail call used in the internet, every msg starts with a word *from:* colon. That would be a prefix that the adversary knows. So...
$$
c\oplus m_{1,...,i}=G(k)|_{1,...,i}
$$
the PRG is cracked!

**Def:** We say that $G:k\longrightarrow\{0,1\}^n$​ is predictable if

$\exist$ "eff" alg. A and $\exist 1\leq i\leq n-1$ s.t.
$$
Pr_{k \overset{R}{\leftarrow}K}[A(G(k)|_{1,...,i})=G(k)|_{i+1}]\geq \frac{1}{2}+\epsilon\ \ \ (1.5)
$$
for some non-neg $\epsilon\geq1/2^{30}$

**Quiz:**

- Is G: XOR(G(k)) = 1 predictable? Yes, given the first (n - 1) bits can predict the n'th bit.

### Neg

In practice: $\epsilon$ is a scalr and 

$\epsilon$ non-neg: $\epsilon\geq1/2^{30}$ likely to happen over 1GB data

$\epsilon$ neg: $\epsilon\leq1/2^{80}$​ won't happen over life of key

In theory: $\epsilon$ is a function $\epsilon:Z^{\geq0}\longrightarrow R^{\geq0}$ and

$\epsilon$ non-neg: $\exist$d: $\epsilon(\lambda)\geq1/\lambda^b$ inf.often ($\epsilon\geq$1/poly, for many $\lambda$​)



### Weak PRGs

Linear Congruential Generator with parameters a, b, and p such that
$$
r[i]\longleftarrow a*r[i-1]+b\ \ \ mod\ p
$$

$$
r[0]\equiv seed
$$

and outputs few bits of $r[i]$​ then i++.

```python
/* glibc GNU C Library */
glibc random():
    r[i] = (r[i-3] + r[i-31]) % 2**32
    output r[i] >> 1
```

In fact, systems like Kerberos version four have used and been beaten by that.

## What is a secure cipher?

### PRG Security Defs

Let $G: k \longrightarrow  \{0, 1\}^n$ be a PRG, our goal is to define what it means that $G(k)$ is “indistinguishable” from  $r$.

Remember that,

$G(k)$ is the output when $k \overset{R}{\leftarrow}K$, and $r$ is the output when $r\overset{R}{\leftarrow}\{0, 1\}^n$.

Is this possible? Btw, the size of $\{0, 1\}^n$ is much much more larger than $G(k)$.

Statistical Tests

statistical test on $\{0, 1\}^n$: an alg $A$ s.t. $A(x$) outputs “0” or “1”, 0 means that the test thinks the input you give looks not random, well 1 means the input *is* random.

Examples:

1. $A(x) = 1$ iff $|$#$0(x) -$ #$1(x)|\leq 10\cdot \sqrt{n}$
2. $A(x) = 1$ iff $|$#$00(x) - \frac{n}{4}|$ $\leq 10 \cdot \sqrt{n}$

More examples:

1. $A(x) = 1$ iff max-run-of-$0(x)$ $\leq 10\cdot \log_2(n)$

$\vdots$

Advantage

Let $G: k \longrightarrow  \{0, 1\}^n$ be a PRG and $A$ a stat. test on $\{0, 1\}^n$, we define:

$$ ⁍ $$

Comment:

Adv close to 1 —— A can dist. G from rand;

Adv close to 0 —— A *cannot* dist. G from rand.

A silly example: $A(x) = 0 \Rightarrow Adv_{PRG}[A, G] = |0-0|=0$

An interesting example:

Suppose $G: k \longrightarrow  \{0, 1\}^n$ satisfies $msb(G(k)) = 1$ for 2/3 of keys in $K$, define stat.test $A(x)$ as:

if $msb(x) = 1$ output “1” else output “0”

Then

$Adv_{PRG}[A, G] = |Pr[A(G(k)=1)]-Pr[A(r)=1]| = |\frac{2}{3}-\frac{1}{2}|=\frac{1}{6}$

$msb$ means the *m*ost *s*ignificant *b*it of the string, the first bit of the output.

this basically means A breaks G with advantage 1/6 .

Secure PRGs: crypto definition

Def: We say that $G: k \longrightarrow  \{0, 1\}^n$ is a secure PRG if

for all “eff” stat. tests A: $Adv_{PRG}[A, G]$ is “neg”

Are there provable secure PRGs? Sorry, we *cannot*! P = NP?

if we can prove P = NP, it’s easy to show there are no secure PRGs;

if P is *not* equal to NP, you could prove to me a particular PRG is secure.

but we have heuristic candidates.

Easy fact: a secure PRG is unpredictable

We show: PRG predictable means PRG is insecure

Suppose A is an eff alg s.t.

$Pr_{k \overset{R}{\leftarrow}K}[A(G(k)|*{1,...,i})=G(k)|*{i+1}]=\frac{1}{2}+\epsilon$

for non-negligible $\epsilon$ e.g. $\epsilon$ = 1/1000

Define stat. test B as:

$B(x)$ outputs “0” or “1”, 1 means that $A(x|*{1,...,i})=x*{i+1}$ .

如果一个PRG是可预测的，那么这个PRG是不安全的。也就意味着其逆否命题——一个安全的PRG一定是不可预测的——是正确的。

假设我们有一个有效的预测算法 A ，使得：

$$ Pr_{k \overset{R}{\leftarrow}K}[A(G(k)|*{1,...,i})=G(k)|*{i+1}]=\frac{1}{2}+\epsilon $$

对于一个真正随机的字符串，已知前 i 位成功预测下一位的概率是1/2 . 这是由于每一位要么是0要么是1. 如果对于算法 A ，能够以一个不可忽略的$\epsilon$，例如$\epsilon = 1/1000$ 成功预测下一位，那么这就是个危险的预测算法。

下面我们定义一个新的统计测试 B ，以字符串 x 作为输入，输出 0 或者 1.如果 A 根据前缀正确预测下一位，即 $A(x|*{1,...,i})=x|*{i+1}$，那么$B(x)=1$.

对于一个真正随机的字符串$r\overset{R}{\leftarrow}\{0, 1\}^n$$Pr[B(r)=1]= \frac{1}{2}$ 成立。换句话说，算法 A 不提供第 i + 1 位的任何信息。

$$ Adv_{PRG}[B, G] > \epsilon $$

如果发生器 G 是安全的，那么不存在有效的算法 A 和好的统计测试 B 以不可忽略的优势赢下游戏。

1982年姚期智在论文 *[Theory and application of trapdoor functions](https://www.di.ens.fr/users/phan/secuproofs/yao82.pdf)* 中用熵的观点给出了一个十分优雅且重要的定理，即上述定理的逆命题也是正确的：如果发生器是不可预测的，那么这个发生器就是安全的。