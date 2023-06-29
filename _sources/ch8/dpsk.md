# $M$-ary Differential Phase Shift Keying ($M$-DPSK) 
* Consider the $M$-PSK constellation 
  $\left\{ e^{j 2\pi m/M}: m=0,1,\ldots,M-1 \right\}$. Let $c[n]$ be
  the information symbol sequence chosen from the $M$-PSK
  constellation, and $b[n] = c[n] b[n-1]$ is the differentially
  encoded symbol sequence. Then the linearly modulated signal 
  $x(t) = \sum_n b[n] p(t-nT)$ gives us $M$-ary DPSK.
* Practically, a known value needs to be set for the initial symbol,
  say, $b[-1] = 1$.
* Consider the ML non-coherent demodulation of $c[0]$. We may treat
  \begin{equation*}
  s_m(t) = b[-1] p(t+T) + b[0] p(t) = b[-1] p(t+T) + e^{j 2\pi m/M} b[-1] p(t)
  \end{equation*}
  in this case. If $p(t)$ has most of its energy concentrated to
  within a symbol period, the signals $s_m(t)$'s have approximately
  equal energies. Hence the ML non-coherent demodulator is given in
  {eq}`e:nc_ml_ee`:
  ```{math}
  :label: e:nc_ml_dpsk
  \begin{align} 
  \hat c[0]
  & = \arg\hspace{-15pt}\max_{m \in \{0,1,\ldots,M-1\}} 
  \left| \tilde r[-1] + e^{-j 2\pi m/M} \tilde r[0] \right| 
  \notag \\ 
  & = \arg\hspace{-15pt}\max_{m \in \{0,1,\ldots,M-1\}} 
  \text{Re} \left(e^{-j 2\pi m/M} \tilde r[0] \tilde r^*[-1] \right)
  \notag \\ 
  & = \arg\hspace{-15pt}\max_{m \in \{0,1,\ldots,M-1\}} 
  \cos \left(\angle \tilde r[0] - \angle \tilde r[-1] - \frac{2\pi m}{M} \right)
  \end{align}
  ```
  where $\tilde r[n] = r(t) * p^*(-t) \big|_{t=nT}$.
* The demodulation rule in {eq}`e:nc_ml_dpsk` is simply finding the
$M$-PSK signal point on the unit circle that is closest to the point
with angle $\angle \tilde r[0] - \angle \tilde r[-1]$.
* The ISI-free property of the RRC pulse discussed before guarantees
that the demodulation rule in {eq}`e:nc_ml_dpsk` is applicable to the
case of multiple TX symbols.
