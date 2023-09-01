# Multi-rate Filtering

* Without exception, we need to filter both the TX and RX signals in
  any communication system. Analog and digital filters are often
  required in the RF frontend and baseband processor, respectively.
* For the USRP radios, analog filtering is performed in the
  daughterboards. In the RX chain of the SBX daughterboard, there is
  an analog lowpass filter of 40 MHz bandwidth for the I-/Q-channel,
  before entering the ADC on the motherboard. Similarly, the analog
  I-/Q-signal generated from the DAC on the motherboard is filtered by
  an analog lowpass filter of 40 MHz bandwidth before being mixed up
  to the TX carrier frequency. For the N210 USRP, digital filtering
  is performed by the DUCs and DDCs implemented in the FPGAs on the
  motherboard.
* Despite all filtering applied at the USRP, we still often need to
  filter the signal samples in the host computer before sending them
  to the USRP for transmission and/or after receiving them from the
  USRP. Typical examples of such filtering operations include **pulse
  shaping (PS)** for TX and **matched filtering (MF)** for RX.
* For both PS and MF, the filters involved are typically FIR filters
  with impulse responses of long lengths. Direct time-domain
  implementations of these long FIR filters are generally not
  computationally efficient. Efficiency is critical to us since we may
  need to implement these filters in the host PC simultaneously with
  many other computationally intensive processing
  functions. Therefore, we may need to opt for frequency-domain
  implementations of the filters with FFT, which will be the main work
  horse of our SDR implementations.
* In addition, we have mentioned before that there are limitations on
  the choice of sampling rate due to the half-band filters in the DDCs
  implemented in the FPGA of the N210 USRP.
  In order to work with these limitations while finely
  controlling the symbol rate and spectral shape of our communication
  signal, we may
  need to do **multi-rate filtering**, where the input and output rates
  may not be the same. As a result, we may have to do *interpolation
  (up-sampling)* and *decimation (down-sampling)* on the signal samples.
* <font color="red">I assume that everyone is familiar with the
  various concepts of DFT, FFT, frequency-domain filtering (in
  particular the overlap-save algorithm), interpolation, decimation,
  and polyphase filtering.</font> Hence, our main focus here will be
  on C++ implementations of these techniques. If you need to review
  these concepts, a classical reference is {cite}`oppenheim2010` Chapters
  4,8, and 9.
