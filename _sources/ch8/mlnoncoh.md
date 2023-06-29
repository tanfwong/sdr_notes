# Maximum Likelihood Non-coherent Demodulation
* Let us begin with the simplifying model that the TX signal is
  $x(t) = s_m(t)$, where $s_m(t)$ is one signal from the $M$-ary signal
  constellation $\{s_0(t), s_1(t), \ldots, s_{M-1}(t)\}$. The RX signal
  is modeled as 
  \begin{equation*}
  r(t) = x(t) e^{j\theta} + n(t)
  \end{equation*}
  where, as before, the uniform random variable $\theta$ models the
  unknown carrier phase and $n(t)$ is AWGN. We have also assumed that
  the long-term (averaged over many symbols) RX signal gain is
  normalized to $1$ by the AGC. Since timing synchronization has been
  achieved, there is no need to model the transmission delay.
* If all the signals in the constellation are equally likely to be
  transmitted, then the ML non-coherent demodulator minimizes the
  average symbol (demodulation) error probability. It is not hard to
  show (see {cite}`proakis2008` Ch. 4) that the ML non-coherent
  demodulator is given as below: 
  ```{math}
  :label: e:nc_ml
  \begin{equation}
  \hat m = \arg\hspace{-15pt}\max_{m \in \{0,1,\ldots,M-1\}} \ln I_0 \left(
  \frac{1}{N_0} \left| \int_{-\infty}^{\infty} r(t) s^*_m (t) dt \right|
  \right) -\frac{E_m}{2 N_0} 
  \end{equation}
  ```
  where $E_m \triangleq \int_{-\infty}^{\infty} |s_m(t)|^2 dt$ is the
  energy of the $m$th signal and $I_0(\cdot)$ is the $0$th order
  modified Bessel function of the first kind.
* In general, we need to know the noise spectral density $N_0$ (after
  the AGC normalization) in order to implement the ML non-coherent
  demodulator.
* However, if all the signals in the constellation are of equal
  energy, the ML non-coherent demodulator reduces to
  ```{math}
  :label: e:nc_ml_ee
  \begin{equation}
  \hat m = \arg\hspace{-15pt}\max_{m \in \{0,1,\ldots,M-1\}}  
  \left| \int_{-\infty}^{\infty} r(t) s^*_m (t) dt \right| 
  = \arg\hspace{-15pt}\max_{m \in \{0,1,\ldots,M-1\}} \left| r(t) * s^*_m(-t) \big|_{t=0} \right|
  \end{equation}
  ```
  because of the strictly increasing nature of $\ln I_0(x)$ for non-negative $x$.
* For linear modulation, the signal constellation is 
  $\{ b_{0} p(t), b_{1} p(t), \ldots, b_{M-1} p(t)\}$, where the pulse shape $p(t)$ has
  (normalized) unit energy. Thus, the ML non-coherent demodulator
  becomes
  ```{math}
  :label: e:nc_ml_lm
  \begin{equation}
  \hat m = \arg\hspace{-15pt}\max_{m \in \{0,1,\ldots,M-1\}} 
  \ln I_0 \left( \frac{|b_m|}{N_0}  \left| \tilde r[0] \right| \right) -\frac{|b_m|^2}{2 N_0} 
  \end{equation}
  ```
  where $\tilde r[n] \triangleq r(t) * p^*(-t) \big|_{t=nT}$ is the MF
  output sampled at time $t=nT$.
