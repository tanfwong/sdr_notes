# USRP Implementation
* The ML non-coherent demodulators for both $M$-ASK and $M$-DPSK use
demodulation statistics that are obtained by sampling the MF (matched
to the TX pulse shape) output at the symbol rate.
* Assume that fine signal timing is obtained by a joint acquisition
and symbol synchronization step at the beginning of the packet. Let
the interpolation and decimation factors of $U$ and $D$ respectively
are needed to convert from the USRP sampling rate to the symbol rate.
* One simple way to generate the sequence of MF output statistics is
to perform MF with an interpolation factor $U$ on the RX samples
from the USRP and then decimate the MF output by a factor of $D$ with the
correct timing obtained from the synchronization step.
* A more efficient way is to perform by directly employing a
$\frac{U}{D}$-rate polyphase overlap-save implementation. Note that
the decimation step takes place at time instants $nD$ in the
interpolated domain. Therefore, additional delay may need to be added
to the MF impulse response to make sure that the decimation timing
aligns with the fine symbol timing obtained from the synchronization
step.

# References and further reading
```{bibliography} 
:style: unsrt
:filter: key % "proakis2008" 
```
