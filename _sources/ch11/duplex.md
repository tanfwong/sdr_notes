# Duplex Transmission using USRPs
* As discussed above, a feedback channel from the sink radio back to
  the source radio is needed in order to support ARQ. That means each
  radio needs to be able to both transmit and receive signals.
* Since each SBX/CBX daughterboard has two independent RF chains (one
  for TX and one for RX), our USRP radios may support full-duplex
  operation. That is, each radio can transmit and receive at the same
  time.
* However, the TX and RX antennas of a USRP radio are often closely
  placed together. Thus the signal transmitted by the TX chain of the
  USRP will be picked up by the USRP's own RX chain in addition to the
  intended RX signal, causing a very strong self interference. The
  signal power of this self interference could be tenths of dB above
  that of the intended RX signal.
* This extremely strong self interference, if cannot be filtered out
  before the ADC, often saturates the ADC in the RX chain, resulting
  in a strong nonlinear additive interference term in the digitized RX
  signal samples at the ADC output. It is usually difficult to remove
  this nonlinear additive interference from the RX signal samples,
  even if the signal transmitted by the USRP causing the interference
  is known.
* One may attenuate the analog signal in the RX chain before the ADC
  enough that the attenuated signal falls within the dynamic range of
  the ADC. However, doing so will also attenuate the intended RX
  signal so much that quantization errors of the ADC becomes too
  significant, even if one can remove the known self interference from
  the RX signal.
* If one is to perform cancellation of the self interference in the RX
  chain, that must be done in the analog domain before the
  ADC. Unfortunately, our USRP radios are not equipped with the
  analogu circuitry to perform self interference cancellation.
* In all, while full-duplex operations in the same frequency band
  using our USRP radios is theoretically possible, it is practically
  hard to do so. Therefore, we may opt for either one of the following
  more practical approaches:

## Frequency-division duplexing (FDD)
* In FDD, the forward and feedback channels occupy different frequency
  bands. Self interference in full-duplex operations is removed by
  analog filtering.
* Recall that in the RX chain of the SBX/CBX daughterboard has a
  fixed 40 MHz analog lowpass filter before the ADC in each of the I &
  Q paths. To employ this analog filter to remove self interference,
  the center frequencies of the forward and feedback bands need to be
  separated by more than 40 MHz plus the larger of the bandwidths of
  the forward and feedback signals. The band separation often needs to
  be larger due to strong sidelobs that may also present in the self
  interference signal due to amplifier nonlinearities.
* If the above frequency band separation requirement is satisfied,
  full-duplex operations can be supported and hence the ARQ protocols
  described above can be directly implemented:
  - At the source (and similarly the sink), both transmit and receive
    objects (and functions) need to be instantiated and ran
    asynchronously on separated threads.
  - The only communication needed between the transmit and receive
    threads are the values of $S$ ($S_{\max}$ and $S_{\min}$ in
    go-back-$n$ and selective-repeat ARQ) and $R$. This can be simply
    achieved for instance at the source by making $S$ an atomic class
    variable in the transmit object and $R$ an atomic class variable
    in the receive object, providing thread-safe access to these class
    variables.
* The timeout required in the ARQ protocols should be set according to
  the specific protocol accounting for the signal generation and
  processing time at the host computer, the host load condition, the
  latency involved in sending samples back and forth between the USRP
  and host, the processing time in the FPGA in the USRP radio,
  transmission time as well as any buffering delays. Usually practical
  timeout values have to be fine-tuned experimentally.
* Timers may be easily implemented using the C++
  [`std::chrono::steady_clock`](http://www.cplusplus.com/reference/chrono/steady_clock/)
  class.
* To minimize processing latency in the host computer, the threads
  that implement critical transmit and receive functions should be set
  to have the highest priority.

## Time-division duplexing (TDD)
* In TDD, transmissions over the forward and feedback channels occur
  in different non-overlapping time slots over the same frequency
  band. Usually additional guard time is added between adjacent time
  slots to allow for a maximum physical propagation delay.
* Since a radio never transmits and receives simultaneously in TDD, it
  only needs to support half-duplex operations. With our USRP radios,
  the independent TX and RX chains help to avoid any delay involved in
  switching between TX and RX chains.
* If synchronized clock sources (e.g. GPS clocks) are available to
  both the source and sink, the time slots can be synchronized and
  scheduled for use by forward and feedback transmissions.
* If no synchronized clock sources are available (as in our case), TDD
  is still possible based on time slots induced by the ARQ
  protocol. For example, the following modified version of the
  stop-and-wait ARQ protocol can be used in a TDD system without
  synchronized clock sources:

### Stop-and-wait ARQ for TDD without synchronized clocks
* **<u>Source</u>:**
  1. Fetch a new data payload from the higher layer if there is one
     available. Otherwise, wait until a new payload is available.
  1. Use $S$ as the packet number to construct a data packet from the
     payload, and send the constructed data packet to the sink.
  1. Wait for an ACK packet to arrive from the sink. If an error-free
     ACK packet carrying an $R > S$ is received before timeout,
     increment $S$ and go back to step 1 to transmit a new data
     packet. If no such ACK packet is received before timeout, go back
     to step 2 to retransmit the current data packet.
* **<u>Sink</u>:**
  1. Wait until a data packet arrives from the source.
  1. If an error-free packet containing an $S = R$ is received from
     the source, release the previously stored payload (if exists) to
     the higher layer, store the data payload of the received packet,
     increment $R$ and immediately send an ACK packet carrying the
     request number $R$ back to source.
  1. Go back to step 1.
* Note that this protocol establishes a master-slave relationship
  between the source and sink. The source acts as the master to control
  transmission timings of both the forward and feedback channels. The
  sink acts the slave responding to the transmission events initiated by
  the source.
* Additional communication between the threads running the transmit
  and receive objects is needed to implement this protocol. For
  example:
  - At the source, after the transmit object sends a data packet, it
    can send a signal to the receive object to tell it to start
    receiving an ACK packet from the sink. Note that because of the
    difference in transmission rates over the Gigabit Ethernet to and
    from the USRP, an additional delay may be needed before the
    transmit object sends the signal to the receive object.
  - At the sink, the receive object may directly call the `send`
    method of the transmit object to send an ACK packet when the
    protocol calls for such an action. Again, additional delay may be
    needed after the `send` method returns before the receive object
    start processing the samples received from the USRP. This delay
    should implemented by dropping samples from the USRP that
    correspond to the ACK packet that it transmits.

