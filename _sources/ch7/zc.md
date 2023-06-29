# Zadoff–Chu Sequences

* If we allowed complex-valued symbols, then **Zadoff-Chu sequences**
  are popular choices for use as signature sequences.

* An $N$-length *root* Zadoff-Chu sequence parameterized by $M$ 
  is defined by
  \begin{equation*}
  x_M[n] = 
  \begin{cases}
  \exp\left( -j \frac{M \pi n^2}{N} \right) & \text{ if } N \text{ is even} \\
  \exp\left( -j \frac{M \pi n(n+1)}{N} \right)& \text{ if } N \text{ is odd}
  \end{cases}
  \end{equation*}
  where $M$ and $N$ are relative primes. 
* Clearly, different phases $Tx_M, \ldots, T^{N-1}x_M$ are also 
  Zadoff-Chu sequences and have the same periodic auto-correlation
  function as the root  Zadoff-Chu sequence $x_M$.

* <u>Properties of Zadoff–Chu sequences:</u>
  1. $|x_M[n]| = 1$ for all $n$.
  1. $\theta_{x_M}[k] = \begin{cases}
        N & \text{ if } k=0 \text{ mod } N\\
        0 & \text{ if } k\neq 0 \text{ mod } N.
        \end{cases}$
  1. Suppose that $N$ is odd, $M_1-M_2$ is relatively prime to $N$,
     and that $x_{M_1}$ and $x_{M_2}$ are two $N$-length Zadoff-Chu
     sequences. Then $\left|\theta_{x_{M_1}, x_{M_2}}[k]\right| =
     \sqrt{N}$.

* Notice that Property 2 says that any Zadoff-Chu sequence has the
  ideal auto-correlation function. 
* From
  [Property 8 of the cross-correlation function](sec:correlations),
  given that the auto-correlation functions of any pair of sequences
  satisfy Property 2 above, the lower bound on the peak magnitude of
  the cross-correlation function between the two sequences is
  $\sqrt{N}$. Hence, Property 3 above states that the two Zadoff-Chu
  sequences also give the best cross-correlation function performance.
        
