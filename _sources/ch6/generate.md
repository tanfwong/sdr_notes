(sec:txpulse)=
# TX Pulse Shaping

## Power spectral density of digitally modulated signal 

* To see how we may control the spectral shape of the TX signal, we
  need to first obtain the power spectral density (PSD) of a digitally
  modulated signal. We will work in the complex baseband here
  throughout.
* Let $b[n]$ be a stationary sequence of random data symbols that we
  wish to transmit. Let $T$ be the symbol period. For a symbol period,
  we transmit one signal from the finite collection $\{s_0(t), s_1(t), \ldots,
  s_{M-1}(t)\}$ to convey one symbol worth of data. We usually call the
  vector-space representation of the collection of signals, or just
  the collection itself, the **signal constellation**. The choice
  of which signal to send is determined by the whole symbol
  sequence $b[n]$ in general. For the $n$th symbol period, denote the
  signal transmitted by $s(t-nT;n)$, where the delay $nT$ accounts for
  transmission time of this signal and the second argument $n$ should be
  interpreted as the signal choice from the constellation as discussed
  for this $n$th symbol period.
* The TX signal is then 
  ```{math}
  :label: e:txsignal
  \begin{equation}
  x(t) = \sum_{n = -\infty}^{\infty} s(t-nT; n). 
  \end{equation}
  ```
* The autocorrelation function of $x(t)$ is $R_{x}(t+\tau,t)
  \triangleq E[x(t+\tau)x^*(t)]$, which is clearly periodic (with
  period $T$) in $t$. Hence, $x(t)$ is a wide-sense cyclostationary
  process.
* Consider an observer located at some random location from the TX
  (not too far away so that the path loss is minimal). The TX signal
  observed by this observer can then be modeled as $x(t-T_d)$, where
  $T_d$ is a random delay uniformly distributed over the interval
  $[0,T]$, independent of $x[n]$. It can be readily shown (see {cite}`proakis2008`
  Section 3.4 and {cite}`madhow2008` Section 2.3) $x(t-T_d)$ is a wide-sense
  stationary process whose autocorrelation function
  ```{math}
  :label: e:Rx
  \begin{equation}
  \label{} \bar R_x(\tau) = \frac{1}{T} \int_0^T R_x(t+\tau,t) dt =
  \frac{1}{T} \sum_{n=-\infty}^{\infty} g_k(\tau -nT),
  \end{equation}
  ```
  where 
  ```{math}
  :label: e:gk
  \begin{equation}
  g_k(\tau) = \int_{-\infty}^{\infty} E[ s(t+\tau;k) s^*(t;0) ] dt.
  \end{equation}
  ```
* The PSD of $x(t)$ will be defined as the PSD of $x(t-T_d)$, both
  denoted by $S_x(f)$ which is the Fourier transform (FT) of $\bar
  R_x(\tau)$. This definition of PSD for the TX signal $x(t)$ makes
  physical sense since its spectrum is measured by some instrument
  located at some distance away from the TX.
* Let the FT of $g_k(\tau)$ be $G_k(f)$. Then
  ```{math}
  :label: e:Sx
  \begin{equation}
  S_x(f) = \frac{1}{T} \sum_{k=-\infty}^{\infty} G_k(f) e^{-j2\pi k T}.
  \end{equation}
  ```
* Note that the same development can be redone by deterministic
  time-averaging with the PSD of $x(t)$ defined as $\lim_{D\rightarrow
  \infty} \frac{\left| X_D(f) \right|^2}{D}$, where $X_D(f)$ is the FT
  of $x_D(t) \triangleq \begin{cases} x(t) & \text{if } -\frac{D}{2}
  \leq t \leq \frac{D}{2} \\ 0 & \text{otherwise} \end{cases}$. The
  PSD defined this way will still have the same
  expression in {eq}`e:Sx` (see {cite}`madhow2008` Section 2.3 for details). 
  This deterministic definition provides us a measurement-based physical 
  meaning for the PSD of $x(t)$.

## Linear modulation and Pulse shaping 
* For a *linear modulation*, $s(t;n) = b[n] p(t)$, where $p(t)$ is
  called the **TX pulse shape**. Thus, the set of possible symbol values
  is  the constellation. From {eq}`e:gk`, 
  \begin{equation*}
  g_k(\tau) = R_b[k] \int_{-\infty}^{\infty} p(t+\tau)p^*(t)dt
  \end{equation*}
  and $G_k(f) = R_b[k] \left|P(f)\right|^2$, where 
  $R_b[k] \triangleq E[b[k]b^*[0]]$ and $P(f)$ is the FT of the TX
  pulse shape $p(t)$. Furthermore, using {eq}`e:Sx`, we have 
  ```{math}
  :label: e:Sx_lin
  \begin{equation} 
  S_x(f) = \left|P(f)\right|^2 \frac{S_b \left( e^{j2\pi fT} \right)}{T}
  \end{equation} 
  ```
  where
  \begin{equation*}
  S_b\left( e^{j2 \pi \hat f} \right) =
  \sum_{k=-\infty}^{\infty} R_b[k] e^{-j2\pi k \hat f}
  \end{equation*}
  is the discrete-time Fourier transform (DTFT) of $R_b[k]$. The quantity
  $\frac{S_b \left( e^{j2\pi fT} \right)}{T}$ can be interpreted as the
  PSD of the data symbol sequence $b[n]$.
* For BPSK, $b[n]$ are i.i.d. symbols selected from $\{-1,+1\}$ with
  equal probabilities. For DBPSK, $b[n] = a[n] b[n-1]$, where $a[n]$
  are i.i.d. symbols selected from $\{-1,+1\}$ with equal probabilities.
  For these two binary (linear) modulations, $R_b[k] = \delta[n]$ and
  $S_b\left( e^{j2 \pi \hat f} \right) = 1$. Therefore, the TX PSD is
  \begin{equation*}
  S_x(f) = \frac{1}{T} \left|P(f)\right|^2.
  \end{equation*}
  The same PSD formula, within a scaling factor, applies to
  MPSK, DMPSK, and MQAM modulations as well.
* For OOK (2-ASK), $b[n]$ are i.i.d. symbols selected from $\{0,1\}$
  with equal probabilities. So $R_b[k] = \frac{1}{4} + \frac{1}{4}
  \delta[n]$ and 
  \begin{equation*}
  S_b\left( e^{j2 \pi \hat f} \right) = \frac{1}{4} +
  \frac{1}{4} \sum_{k=-\infty}^{\infty} \delta( \hat f - k).
  \end{equation*}
  Therefore, the TX PSD is 
  \begin{equation*}
  S_x(f) = \frac{1}{4T} \left|P(f)\right|^2 +
  \frac{1}{4T} \left|P(f)\right|^2 \sum_{k=-\infty}^{\infty} \delta
  \left( f - \frac{k}{T} \right).
  \end{equation*} 
  Note that the PSD of OOK contains spectral lines.
* For the calculation of the PSDs of more complicated linear
  modulations, see {cite}`proakis2008` Section 3.4. 
* In general, we see from the simple examples above or from {eq}`e:Sx_lin` 
  that the FT of the TX pulse shape $p(t)$ controls the spectral shape of any
  linearly modulated TX signal.
* It is also easy to see that for any linear modulation, the TX signal
  equation in {eq}`e:txsignal` simplifies to 
  \begin{align*} 
  x(t) &= \sum_{n=-\infty}^{\infty} b[n] p(t-nT) \\
  & =
  \left[ \sum_{n=-\infty}^{\infty} b[n] \delta(t-nT) \right] * p(t). 
  \end{align*}
  Hence, we can generate the TX signal $x(t)$ by first modulating an
  impulse train of period $T$ with the symbol sequence $b[n]$ and then
  passing the symbol-modulated impulse train through a filter whose
  impulse response is the pulse shape $p(t)$. This method of generating the
  linearly modulated signal is called **pulse shaping**.

## Root raised cosine pulse 
* A popular choice of pulse shape that allows us to easily tune the
  spectral shape is the **root raised cosine (RRC) pulse** 
  defined by the expression below:
  ```{math}
  :label: e:rrc
  \begin{equation} 
  p_{RRC}(t) = \frac{\sin \frac{\pi (1-\beta) t}{T} + \frac{4\beta t}{T} \cos \frac{\pi
  (1+\beta) t}{T}}{ \frac{\pi t}{T}
  \left[ 1 - \left(\frac{4\beta t}{T} \right)^2 \right] } 
  \end{equation}
  ```
  where the parameter $0 \leq \beta \leq 1$ is called the **excess
  bandwidth** or **roll-off factor**. The FT of the RRC pulse is the
  RRC spectrum given by
  ```{math}
  :label: e:RRC_FT
  \begin{equation} 
  P_{RRC}(f) =
  \begin{cases} \sqrt{T} & \text{for } |f| \leq \frac{1-\beta}{2T} \\
  \sqrt{\frac{T}{2} \left\{ 1 - \sin
  \left[\frac{\pi T}{\beta} \left( |f| - \frac{1}{2T} \right) \right]
  \right\} } & \text{for } \frac{1-\beta}{2T} < |f| \leq
  \frac{1+\beta}{2T} \\ 0 & \text{for } |f| > \frac{1+\beta}{2T}.
  \end{cases} 
  \end{equation}
  ```
* We see from {eq}`e:RRC_FT` that the FT of the RRC pulse
has a passband from $-\frac{1+\beta}{2T}$ to $\frac{1+\beta}{2T}$ Hz.
The band edge becomes steeper as $\beta$ gets closer to $0$, at which
the FT becomes that of an ideal LPF. The support of the RRC pulse is
infinite, but its signal magnitude has a $\frac{1}{t^2}$ decay (except
when $\beta=0$). In general, the closer $\beta$ is to $0$, the longer
is the essential support of the RRC pulse.


## USRP implementation
* Typically, pulse shaping is implemented in discrete-time in the host
computer using a multi-rate filter. The filter output samples are then
sent to the USRP for transmission.
* The input to the multi-rate filter should be the symbol sequence
$b[n]$ at the symbol rate $\frac{1}{T}$, while the output contains the
signal samples to be sent to the USRP at the TX sampling rate.
* The impulse response of the multi-rate filter (in the upsampled
domain) is $h[n] = p\left( \frac{nT}{U} \right)$, where $U$ is the
interpolation factor of the multi-rate filter.
* We will use the RRC pulse $p_{RRC}(t)$ to perform pulse shaping.
Because the RRC pulse has an infinite support, we have to approximate
the ideal RRC pulse-shaping filter by using a truncated RRC pulse as
the impulse response $h[n]$.
* In general, the closer is the excess bandwidth $\beta$ to $0$ a
longer $h[n]$ is needed to accurately approximate the ideal RRC
pulse-shaping filter. In addition, the expression in {eq}`e:rrc` for
$p_{RRC}(t)$ gives rise to a non-causal filter. Hence we need to delay
$p_{RRC}(t)$ and then sample to obtain a causal $h[n]$. The usual
choice is to delay $p_{RRC}(t)$ such that the peak of the RRC pulse
(at $t=0$) appears at the middle sample of $h[n]$. Remember this delay
in the impulse response will introduce a delay in the generated $x(t)$
by the same amount of time.
* Let $f_s$ be the sampling rate of the USRP. Since the passband of
the RRC pulse is from $-\frac{1+\beta}{2T}$ to $\frac{1+\beta}{2T}$
Hz, the sampling theorem tells us that we must have
$f_s \geq \frac{1+\beta}{T}$ or equivalently $\frac{1}{T} \leq
\frac{f_s}{1+\beta}$ if we are to perfectly construct $x(t)$ from the
output samples of the multi-rate filter. Thus we would want to set
$\frac{1}{T} = \frac{f_s}{1+\beta}$ and choose $\beta$ as close to $0$
as possible so that we can maximize the symbol rate. However, recall
that $h[n]$ needs to be long when $\beta$ is close to $0$. Moreover,
the upsampling and downsampling factors $U$ and $D$ are related to
$f_s$ and the symbol rate $\frac{1}{T}$ as $\frac{U}{D} = f_s T$.
Hence, we need to set $\frac{U}{D} = 1 + \beta$. This implies that we
will need to use larger integers for $U$ and $D$ to make $\beta$ close
to $0$. Such large choices of $U$ and $D$ will undesirably increase
the complexity of the multi-rate filter.
* Practically we would choose small integers $U > D$ such that
  $\frac{U}{D} -1$ is relatively small. Then we choose $\beta =
  \frac{U}{D} -1$ and $\frac{1}{T} = \frac{D}{U} f_s$. For examples:
  - Choosing $U=4$ and $D=3$ gives $\beta = \frac{1}{3}$ and achieves a symbol rate of $\frac{3}{4} f_s$.
  - Choosing $U=5$ and $D=4$ gives $\beta = \frac{1}{4}$ and achieves a symbol rate of $\frac{4}{5} f_s$.
  - Choosing $U=8$ and $D=7$ gives $\beta = \frac{1}{7}$ and achieves a symbol rate of $\frac{7}{8} f_s$.
* Below is a simple C++ function to generate the truncated RRC impulse response
  based on the design above:
  ```c++
  /* Generate a unit-energy truncated RRC impulse response for pulse shaping
      Assume D <= U <= 2*D
      This RRC pulse is sampled at U*symbol_rate.
      Excess bandwidth = U/D - 1
      sampling rate / symbol rate =  U/D
      Length of impulse response = 2*len+1
  */
  void rrc_pulse(std::complex<float>* h, int len, int U, int D)
  {
     float beta = float(U - D)/D; //roffoff factor
      h[len] = 1.0-beta+4.0*beta/M_PI;
      float scale = std::norm(h[len]);
      for (int n=1; n<=len; n++) {
          if (n == U/beta/4.0) {
              h[len+n] = beta/sqrt(2.0)*((1.0+2.0/M_PI)*sin(M_PI/4.0/beta)+(1.0-2.0/M_PI)*cos(M_PI/4.0/beta));
          } else {
              h[len+n] = (sin(n*M_PI*(1.0-beta)/U) + 4.0*n*beta/U*cos(n*M_PI*(1.0+beta)/U))*U/n/M_PI/(1.0-16.0*n*n*beta*beta/U/U);
          }
          h[len-n] = h[len+n];
          scale += 2.0*std::norm(h[len+n]);
      }
      scale = sqrt(scale);
      for (int n=0; n<2*len+1; n++) {
          h[n] /=  scale;
      }
  }
  ```

* Remember the TX amplifier on the daughterboard of the USRP is
non-linear in the high-gain region. Nonlinearity of the amplifier will
introduce spurious emission outside the intended band. Therefore, the
TX amplifier gain should not be set to too high if linear operation is
required. By default the range of $[-1.0, 1.0]$ of the signal samples
corresponds to the full dynamic range of the DAC on the motherboard of
the USRP. Thus, linearity can be further assured by reducing the
maximum amplitude of the signal samples sent to the USRP.
