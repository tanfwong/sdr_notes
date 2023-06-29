# $M$-ary Amplitude Shift Keying ($M$-ASK) 
* From the form of the ML non-coherent demodulator in
  {eq}`e:nc_ml_lm`, we see that any linear modulation that can be
  non-coherently demodulated must have distinct $|b_m|$ in the
  constellation. If all $b_m$s are real, this results in ASK.

* We may assume $0 \leq b_0 < b_1 < \cdots < b_{M-1}$. For high SNR
  ($N_0 \ll 1$), the ML non-coherent demodulator in {eq}`e:nc_ml_lm`
  may be approximated by 
  \begin{equation*}
  \hat m = \arg\!\max_{m \in \{0,1,\ldots,M-1\}} b_m
  \left| \tilde r[0] \right| - \frac{|b_m|^2}{2}
  \end{equation*}
  which is equivalent
  to comparing the statistic $\left| \tilde r[0] \right|$ to a sequence
  of thresholds $0 = a_0 < a_1 < a_2 < \cdots < a_{M-1} < a_{M} =
  \infty$ to get $\hat m = m$ if $a_m \leq \left| \tilde r[0] \right| <
  a_{m+1}$. It is easy to see that $a_m = \frac{b_{m-1}+b_m}{2}$ for
  $m=1,2,\ldots,M-1$.
* Fixing the average signal energy in the constellation, we may obtain
  the optimal ASK constellation by minimizing the average symbol error
  probability 
  \begin{equation*}
  \frac{1}{M} \sum_{m=0}^{M-1} \Pr \left\{ \left| \tilde r[0] \right| \notin
  [a_m,a_{m+1}) \big| m \right\}. 
  \end{equation*}
  The optimal ASK constellation is one that gives equal conditional
  symbol error probabilities. In general, this optimization problem
  is rather hard to solve. For the special case of $M=2$ (also called OOK),
  the optimal choice is clearly $b_0 =0$ and $b_1 = \sqrt{2}$ if the 
  average symbol energy fixed at $1$.
* If the RRC pulse shape is used, it can be shown (see 
  {cite}`proakis2008` Ch. 9) that $p_{RRC}(t-nT) * p^*_{RRC}(-t) \big|_{t=0} = 0$
  for all $n \neq 0$. This result, which is usually referred to as the *ISI-free* 
  property of the RRC pulse, implies that adjacent symbols play no role in the
  decision statistic of the current symbol when fine timing synchronization is achieved.
  Hence the ML non-coherent demodulator discussed above directly applies to 
  the practical ASK signal containing multiple symbols.

