# Simple ARQ Protocols
* Assume that: 
  - There are a data source radio and a data sink radio. The data
    source sends data packets to the data sink while the data sink
    sends feedback (ACK) packets to the data source. So both radios
    must be capable of transmitting and receiving packets.
  - Each data packet transmitted by the source contains a prefix, a
    header that includes the packet (sequence) number $S$, a data
    payload, and a CRC checksum on the concatenation of the header and
    data payload.
  - Each ACK packet transmitted by the data sink contains a prefix, a
    header that includes a request number $R$, a data payload (if
    needed), and a CRC checksum on the header and data payload.
  - The CRC checksums are able to provide perfect error detection
    capability.
  - The source and sink are properly initialized at the beginning of
    the data transmission process. That is, $S = R = 0$ and no packet
    has been sent by either the source or the sink.
* If there is an information flow in the direction from the sink radio
  to the source radio, regular packets in that flow can double as ACK
  packets described above. In this case, the packet header should
  contain the packet number of that flow and the request number of the
  feedback flow.
* The steps of three basic ARQ protocols are described below for easy
  reference. To better understand them, please study the timing
  diagrams and proofs of correctness of these protocols in
  {cite}`bertsekas1992` Ch. 2.

## Stop-and-wait ARQ
* **<u>Source</u>:**
  1. Fetch a new data payload from the higher layer if there is one
     available. Otherwise, wait until a new payload is available.
  1. Use $S$ as the packet number to construct a data packet for the
     payload, and send the constructed data packet to the sink.
  1. If an error-free ACK packet is received back from the sink
     carrying an $R > S$, then increment $S$ and go back to step 1 to
     transmit a new data packet. If no such ACK packet is received
     within a pre-determined delay (**timeout**), go back to step 2 to
     retransmit the current data packet.
* **<u>Sink</u>:** Initialize a timer. Repeatedly implement the following
  two steps asynchronously:
  1. Wait until an error-free data packet containing an $S = R$ is
     received from the source. Release the data payload of the
     received packet to the higher layer, increment $R$, reset the
     timer, send an ACK packet carrying the request number $R$ back to
     source.
  1. If the timer times out, reset the timer and send an ACK packet
     carrying the request number $R$ back to source.

## Go-back-$n$ ARQ 
* **<u>Source</u>:** Initialize $S_{\min} = S_{\max} = 0$ and a
  timer. Repeatedly implemented the following three steps
  asynchronously:
  1. If $S_{\max} < S_{\min} + n$, fetch a new data payload (if
     available), set $S = S_{\max}$ as the packet number to construct
     a data packet for the payload, increment $S_{\max}$, and send the
     constructed packet to the sink. Store this packet until
     $S_{\min}>S$.
  1. If $S_{\min} < S_{\max}$ and no transmission is occurring, choose
     $S = S_{\min}$ if the timer times out; otherwise choose any $S$
     in the range $S_{\min} \leq S < S_{\max}$. If the choice of $S$
     is $S_{\min}$, reset the timer.  Resend the stored data packet
     with packet number $S$ to the sink.  We may choose $S_{\min} \leq
     S < S_{\max}$ (if timer hasn't expired) in any way. For example,
     we may $S$ cycle through the possible values.
  1. If an error-free ACK packet is received back from sink carrying an
    $R > S_{\min}$, set $S_{\min} = R$ and reset the timer.
* **<u>Sink</u>:** Same as stop-and-wait ARQ.

## Selective-repeat ARQ
* Assume a window size of $n$ as in go-back-$n$ ARQ.
* **<u>Source</u>:** Same as go-back-$n$ except that the additional
  information obtained from the ACK packets may be used to judicially
  select the retransmission packet in step 2.
* **<u>Sink</u>:** Initialize a timer. Repeatedly implement the following
  two steps asynchronously:
  1. Wait until an error-free data packet containing an $S$ in the
     range $R \leq S < R+n$ is received from the source. Store the
     payload of this received packet. If all the payloads of packets
     with packet numbers between $R$ and $S$, inclusively, are
     received and in storage, release all these payloads
     (sequentially) to the higher layer, set $R = S+1$, reset
     the timer, and send an ACK packet with request number $R$ (and
     other additional information if any) back to the source.
  1. If the timer times out, reset the timer and send an ACK packet
     carrying the request number $R$ (and other additional information
     if any) back to the source. 

  The additional information in the ACK packets can be the packet
  numbers $> R$ of some or all of the packets (or haven't been)
  successfully received and in storage within the sliding window. The
  source may use this information to determine its retransmission
  strategy.

