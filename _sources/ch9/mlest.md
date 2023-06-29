# Basic Model
* Suppose that the linearly modulated signal $x(t) = \sum_n b[n]
  p(t-nT)$ is transmitted. Further assume that the TX pulse shape
  $p(t)$ satisfies the ISI-free property, like the RRC pulse.
* Assuming that timing synchronization is achieved, the RX signal can
  be modeled by
  \begin{equation*}
  r(t) = x(t) e^{j(\omega_o t + \theta)} + n(t)
  \end{equation*}
  where $n(t)$ is AWGN as before. The deterministic unknown constants
  $\omega_o$ and $\theta$ model the carrier frequency and phase
  offsets of the LO w.r.t. the carrier in the RX signal,
  respectively. We also assume that the oscillators used in the TX and
  LO are sufficiently accurate so that $|\omega_o| \ll
  \frac{2\pi}{T}$. We are to jointly estimate (or track) $\omega_o$
  and $\theta$.

# ML Estimation Using Pilot Symbols
* Suppose first that $b[n]$ is known to the RX. That is, they are all
  pilot symbols. This can be the situation in which we reuse the
  signature sequence for acquisition and/or timing synchronization as
  pilots to perform carrier synchronization. Under this condition, it
  can be shown that the joint ML estimator for $(\omega_o, \theta)$ is 
  ```{math}
  :label: e:dd_ml
  \begin{align} 
  (\hat \omega_o, \hat \theta)
  &= \arg\!\max_{\!\!\!\!\omega_o, \theta} \text{Re} \left\{
  \int_{-\infty}^{\infty} r(t) x^*(t) e^{-j(\omega_o t + \theta)} dt
  \right\} \label{e:dd_ml_con} 
  \\ 
  & \approx
  \arg\!\max_{\!\!\!\!\omega_o, \theta} \text{Re} \left\{ \sum_n \hat
  r[n] e^{-j(\omega_o nT+\theta)} \right\} 
  \end{align}
  ```
  where
  \begin{equation*}
  \hat r[n] = b^*[n]
  \left[ r(t) * p^*(-t) \big|_{t=nT} \right].
  \end{equation*}
  Note that the approximated ML estimator above is obtained based on
  the assumption that $\omega_o \ll \frac{2\pi}{T}$.
* It is clear that the (approximate) ML estimator in {eq}`e:dd_ml` can
  be rewritten as
  ```{math}
  :label: e:dd_mf_freq
  \begin{equation}
  (\hat \omega_o, \hat \theta) 
  = \arg\!\max_{\!\!\!\!\omega_o, \theta} \text{Re} 
  \left\{ e^{-j\theta} \hat R( e^{j\omega_o T} ) \right\}
  \end{equation}
  ```
  where 
  \begin{equation*}
    \hat R( e^{j\hat\omega}) 
    \triangleq \sum_n \hat r[n] e^{-j\hat\omega n}
  \end{equation*}
  is the DTFT of $\hat r[n]$. The decision rule in {eq}`e:dd_mf_freq` is
  equivalent to
  ```{math}
  :label: e:dd_ml_f
  \begin{align} 
  \hat \omega_o 
  &= \frac{1}{T} \arg\!\!\max_{\!\!\!\!\hat \omega \in
  [-\pi, \pi)} \left| \hat R(e^{j\hat\omega}) \right| 
  \notag \\ 
  \hat \theta &= \angle \hat R(e^{j\hat \omega_o T}).
  \end{align}
  ```
* We may also periodically insert pilot symbols into the data
  symbol sequence. Because of the ISI-free property of the TX pulse, we 
  may simply ignore the data symbols and consider only the pilot symbols
  as far as estimating the carrier frequency and phase goes. For example,
  if a pilot symbol is inserted every $P$ symbol periods, the ML 
  estimator is then:
  ```{math}
  :label: e:dd_ml_f_P
  \begin{align} 
  \hat \omega_o 
  &= \frac{1}{PT} \arg\!\!\max_{\!\!\!\!\hat \omega \in [-\pi, \pi)} 
  \left| \hat R_P(e^{j\hat\omega}) \right| 
  \notag \\ 
  \hat \theta 
  &= \angle \hat R_P(e^{j\hat \omega_o T}). 
  \end{align}
  ```
  where 
  \begin{equation*}
  \hat R_P( e^{j\hat\omega}) \triangleq \sum_n \hat r[nP] e^{-j\hat\omega n}.
  \end{equation*}
  The form of $\hat \omega_o$ clearly requires $|\omega_o| <
  \frac{\pi}{PT}$ for the joint ML estimator to work. Thus the rate of
  pilot symbols must also satisfy the Nyquist criterion corresponding to 
  the maximum possible range of carrier frequency offset.
* <u>USRP implementation:</u> 
  - We may collect the symbol-rate samples of the MF (matched to the
    TX pulse shape) output corresponding to the pilot symbols, and
    then employ FFT to obtain a frequency-domain sampled version of
    the DTFT of the collected MF output to implement the joint ML
    estimator in {eq}`e:dd_ml_f` or {eq}`e:dd_ml_f_P`.
  - If the packet is short such that $\omega_o$ and $\theta$ remain
    constant within the packet, then it suffices to perform a one-shot
    ML estimation of $\omega_o$ and $\theta$ using the joint ML
    estimator in {eq}`e:dd_ml_f` or {eq}`e:dd_ml_f_P`. Then, we may
    correct the frequency and phase offsets by multiplying the properly
    timed MF output $\tilde r[n] = r(t) * p^*(-t) \big|_{t=nT}$ with 
    $e^{-j(\hat\omega_o nT + \hat\theta)}$ before doing coherent
    demodulation for the data symbols.
