# Automatic Repeat Request
* **Automatic Repeat Request (ARQ)** is the generic term for a class
  of error control strategies employed in the data-link control (DLC)
  layer.
* Typically, ARQ implies retransmission of erroneous packets in the
  DLC layer. In order to do so, ARQ requires the following two basic
  capabilities of the underlying communication system:
  1. The ability to detect transmission errors in the received packets. 
  1. The availability of a feedback channel from the TX back to the RX to
     signal the need of retransmission.

* We will introduce a common class of error detecting codes, called
  the **cyclic redundancy check (CRC)** codes, to perform error
  detection.
* We will also present a few  simple ARQ protocols to guarantee
  successful transmission of packets in the DLC layer from
  communication node to another. 
