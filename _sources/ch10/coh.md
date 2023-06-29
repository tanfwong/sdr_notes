# Coherent Demodulation 

After both timing and carrier synchronization have been achieved, we
may perform coherent demodulation.

## Maximum Likelihood Coherent Demodulation

* As in the non-coherent case, we begin with the simplifying model in
  which the TX signal is $x(t) = s_m(t)$, where $s_m(t)$ is one out of
  the $M$-ary signal constellation $\{s_0(t), s_1(t), \ldots,
  s_{M-1}(t)\}$. The RX signal can now be modeled as
  \begin{equation*}
  r(t) = x(t) + n(t)
  \end{equation*}
  where $n(t)$ is AWGN. We have also assumed that the long-term
  (averaged over many symbols) RX signal gain is normalized to $1$ by
  the AGC. Since both timing and carrier synchronization have been
  achieved, there is no need to model the transmission delay and carrier
  offsets.
* If all the signals in the constellation are equally likely to be
  transmitted, then the ML coherent demodulator minimizes the average
  symbol (demodulation) error probability. It is not hard to show (see
  {cite}`proakis2008` Ch. 4) that the ML coherent is given by
  ```{math}
  :label: e:ml
  \begin{align}
  \hat m 
  &= \arg\hspace{-15pt}\max_{m \in \{0,1,\ldots,M-1\}} 
  \text{Re} \left\{ \int_{-\infty}^{\infty} r(t) s^*_m (t) dt \right\} -\frac{E_m}{2} 
  \\
  & = arg\hspace{-15pt}\max_{m \in \{0,1,\ldots,M-1\}} 
  \text{Re} \left\{ r(t) * s^*_m(-t) \big|_{t=0} \right\} -\frac{E_m}{2} 
  \end{align}
  ```
  where $E_m \triangleq \int_{-\infty}^{\infty} |s_m(t)|^2 dt$ is the
  energy of the $m$th signal.
  
* For linear modulation, the signal constellation is $\{ b_0 p(t), b_1
  p(t), \ldots, b_{M-1} p(t)\}$ where the TX pulse shape $p(t)$ has
  (normalized) unit energy. Consider the transmission of a sequence of
  symbols $b[n]$, each chosen from the signal constellation. The TX
  signal is then $x(t) = \sum_{n} b[n] p(t-nT)$. If the TX pulse
  $p(t)$ is ISI-free, the ML coherent demodulator for the $n$th TX
  symbol $b[n]$ becomes
  ```{math}
  :label: e:ml_lm
  \begin{align}
  \hat b[n]
  &= \arg\hspace{-15pt}\max_{m \in \{0,1,\ldots,M-1\}} 
  \text{Re} \left\{ b_m^*\tilde r[n] \right\} -\frac{|b_m|^2}{2} 
  \\
  & = 
  \arg\hspace{-15pt}\min_{m \in \{0,1,\ldots,M-1\}} \left| \tilde r[n] -b_m \right|^2 
  \end{align}
  ```
  where $\tilde r[n] \triangleq r(t) * p^*(-t) \big|_{t=nT}$ is the MF
  output sampled at time $t=nT$.
* Common linear modulations for coherent demodulation are:
  - $M$-PSK, where $b_n \in \left\{ e^{j 2\pi k/M}: k=0,1,\ldots,M-1 \right\}$, and 
  - $M^2$-QAM, where $b_n \in \left\{ \pm \frac{2k+1}{2M} \pm j \frac{2l+1}{2M}: k,l = 0,1,\ldots,M-1\right\}$.

## USRP implementation
* For a packetized implementation with short packets, after the
carrier synchronization step, the symbol-rate MF output samples are
frequency and phase corrected by the simple complex rotator $e^{-j
(n\hat \omega_o T + \hat\theta)}$ to obtain $\tilde r[n]$ as discussed
before. If a PLL is employed to continuously track the carrier phase
function, the carrier correction $e^{-j n\hat \phi[n]}$ may be
applied.
* Then the minimum Euclidean distance decision rule in {eq}`e:ml_lm`
is usually employed to implement the coherent demodulation step.
* For $M$-PSK, the minimum Euclidean distance rule reduces to simply
finding the $M$-PSK signal point on the unit circle that is closest to
the point with angle $\angle \tilde r[n]$.

## References and further reading
```{bibliography} 
:style: unsrt
:filter: key % "proakis2008" 
```
