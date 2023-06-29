# Signal Acquisition

* The objective of signal acquisition is twofold:
  1. To identify whether the captured signal is a signal of interest.
  2. To synchronize to the beginning of the packet. If a DLL is to run
  after acquisition, then only coarse synchronization to an accuracy
  of $\pm \frac{T}{2}$ needs to be achieved. On the other hand, if no
  DLL is to run after acquisition, one may also perform one-shot fine
  symbol synchronization together with acquisition.

* A special sequence of known symbols, called the
  **acquisition/signature/prefix code sequence**, is usually added in
  between the preamble (for the AGC to settle) and the data payload of
  each packet to allow the RX to perform acquisition. In particular, we
  will consider the simple case that the signature signal is a linearly
  modulated signal 
  ```{math}
  :label: e:signature
  \begin{equation}
  x(t) = \sum_{n=0}^{N-1} x[n] p(t-nT)
  \end{equation}
  ```
  where $x[n]$ is the signature sequence and $p(t)$ is the TX pulse
  shape.

## Generalized likelihood acquisition
* Detection of the signature signal is the same as the
  [signal capture](sec:capture) problem that we discussed before, except
  that the whole acquisition signal $x(t)$ is known to the RX and hence may be
  used for acquiring the RX signal. 
* In particular, we may model the RX signal w.r.t. the transmission of
  $x(t)$ as 
  \begin{equation*}
  r(t) = x(t-\tau) e^{j\theta} + n(t)
  \end{equation*}
  where $\theta$ is uniformly distributed over $[-\pi,\pi]$ and $n(t)$
  is an AWGN process as before. The delay $\tau$ is deterministic but
  unknown. The generalized LRT may be used to solve this acquisition
  problem:
  - The ML estimate of $\tau$ is given by {eq}`e:tau`.
  - The generalized LRT is equivalent to compare the statistic
    ```{math}
    :label: e:glrt
    \begin{equation}
    \max_{\tau} \left| r(t) * x^*(-t) \big|_{t=\tau} \right| 
    = \max_{\tau} \left| \sum_{n} x^*[n] \tilde r(nT+\tau) \right|
    \end{equation}
    ```
    to a properly chosen decision threshold.
   - If the threshold is exceeded, the signature signal is acquired
     and the ML estimate $\hat\tau$ gives us the timing of the 
     beginning of the packet.


## USRP implementation
* In practice, we often approximate the generalized LRT by
continuously comparing the magnitude (or squared magnitude) of the MF
output, i.e., $r(t)*x^*(-t)$ in {eq}`e:glrt`, to a threshold
until the threshold is exceeded. The ML estimate $\hat\tau$ is then
searched over a small range beyond the delay at which the threshold is
exceeded. Depending on the ratio of the sampling rate to the symbol
rate (i.e., $f_sT$) and whether fine symbol synchronization is
required in the acquisition step, we may need to implement the MF with
interpolation.

* If short packets are employed, the clocks in the USRP radios are
accurate enough that there is usually no need to run a DLL throughout
the whole packet at the RX. Instead, we may perform fine symbol
synchronization only once, together with acquisition, using the
signature signal directly. In this case, a larger interpolation factor
is needed for the MF in {eq}`e:glrt` to obtain a fine resolution for
the ML estimate $\hat\tau$.
