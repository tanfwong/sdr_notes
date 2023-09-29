# Signal Acquisition and Synchronization

* After we capture the TX signal at the RX, the next step is to
  determine whether the captured signal is of interest and if so
  synchronize to the symbol timing of the captured signal for
  demodulation later.
* These two functions are often achieved in two steps:
   - **Acquisition:** This is the step to determine of the captured
    signal is of interest and establish a coarse synchronization to
    the start of a packet.
  - **Symbol synchronization:** This is a fine synchronization step
    that aligns the RX clock to the symbol boundary of the RX
    signal. The ideal performance is to archive a zero timing error
    modulo the symbol period.
* Acquisition is important since the packet may contain different
  types of symbols organized in larger units, such as a packet header
  and data payload, for different purposes. The acquisition step
  allows us to synchronize to the boundaries between different units
  so that we may process the relevant information. In addition,
  because of how acquisition is commonly implemented, it also serves
  to identify whether the captured signal is one of our interest.
* In a typical digital communication system, acquisition is only
  performed once after the packet's signal is captured. If the packet
  is long, symbol synchronization is achieved and maintained by
  closed-loop circuitry during the whole duration of the packet to
  account for the possibility of hardware clock frequency offset and
  drift.  On the other hand, if the packet is short, one-shot symbol
  synchronization may be performed at the beginning together with
  acquisition.


