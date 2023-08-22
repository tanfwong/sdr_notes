# Introduction

## Typical Radio Architectures
### Transmitter (TX):
```{image} ../figures/tx_arch.png
:alt: TX architecture
:width: 800px
:align: center
```
### Direct down conversion receiver (RX):
```{image} ../figures/rx_ddc.png
:alt: RX direct down-conversion architecture
:width: 800px
:align: center
```
### Superheterodyne receiver (RX):
```{image} ../figures/rx_superhet.png
:alt: RX superheterodyne architecture
:width: 800px
:align: center
```

## Software-defined Radio (SDR)
* Replace the "digital ASIC" components above by more programmable
  implementations, such as FPGA and general purpose CPU/GPU
* In some more recent SDRs, the RF frontends are also made programmable
  to various degrees
* <u>Advantages</u>: Flexible, shorter development time,
  highly reconfigurable (cognitive radios), high-speed implementation
  possible with FPGAs, ...
* <u>Disadvantages</u>: More expensive, higher power consumption, ...
* <u>[Popular commercially available SDRs](https://wiki.gnuradio.org/index.php/Hardware#Commercially_Available_SDR_Platforms)</u>
* <u>Popular SDR software development platforms</u>:
  [GNU Radio](https://www.gnuradio.org/),
  [MATLAB/Simulink](https://www.mathworks.com/hardware-support.html?q=SDR&page=1),
  [Labview](https://www.ni.com/documentation/en/labview/latest/supported-hardware/software-defined-radio/),
  ...

## USRP Radios
* USRP stands for "Universal Software Radio Peripheral"
* A collection of relatively low-cost, high-performance SDRs developed
  by [Ettus Research](http://www.ettus.com/)
* A USRP radio generally consists of a "motherboard" and a number of
  interchangeable "daughterboards" (except for the B-series, 
  E-series, and some of the newer N-series radios)
* A daughterboard typically contains an RF frontend with programmable
  mixers (carrier frequency) and amplifiers (gains)
* The motherboard typically contains high-speed ADCs, DACs, an FPGA,
  and other interfacing and controlling circuitry to communicate with
  a general purpose *host* PC via USB, Ethernet, or PCI bus
* One may program (in Verilog/VHDL) the FPGA on the motherboard to perform
  all IF and baseband processing
* Alternatively the more "software-defined" approach is to have the
  FPGA programmed to perform digital frequency mixing (the mixers in
  the daughterboard have only a finite set of mixing frequencies;
  hence need to do fine-frequency mixing digitally), up/down
  conversion, filtering, and transfer of digital samples to the
  PC. All baseband processing is done by software in the host PC. We will
  employ this latter approach here.

## [USRP N210 Radios](https://www.ettus.com/all-products/un210-kit/) used in class
### Motherboard
```{image} ../figures/n200.png
:alt: N200/210 block diagram from Ettus
:width: 800px
:align: center
```
- DAC/ADC resolution: 16/14 bits
- Maximum transfer rate to/from host host PC: 25 M complex samples/s (16-bit) or 50 M complex samples/s (8-bit)
- Digital up/down convertors:
    * Needed to reconcile the differences between host sampling rates and ADC/DAC sampling rates
    * Host sampling rates should be selected carefully to work with various rate conversion operations and filtering in DDC/DUC
```{image} ../figures/dudc.png
:alt: DDC-DUC
:width: 800px
:align: center
```
### Daughterboards
* <u>SBX:</u>
    - 2 quadrature RF chains: one for TX/RX and one for RX only (RX2)
    -  Each RF chain has its own LO and frequency synthesizer; hence capable of supporting full-duplex operation
    - Programmable TX & RX amplifier gains: 0 - 31.5 dB with 0.5 dB steps
    -  Analog bandwidth: 40 MHz (TX & RX)
    -  Noise figure: 5dB typical
    - Tunable frequency range: 400 MHz - 4.4 GHz
    - TX power: 30 - 100 mW typical
* <u>CBX:</u>
    -  Same as SBX, except having frequency range from 1.2 - 6.0 GHz
* <u>Basic TX and RX:</U>
    - No RF chain, serves as a simple interface between motherboard and external RF frontends (I & Q channels)
    - Analog BW: 250 MHz
    - Can be used (via direct connection to antenna and aliasing) as a
      low-sensitivity RX frontend around the FM band



## [UHD](https://kb.ettus.com/UHD)
* UHD = USRP Hardware Driver
* A set of C++ APIs for controlling USRP radios and pushing/pulling samples to/from them
* Newer versions of UHD also provide C and Python APIs
* Newer versions of UHD also contain [RF Network-on-Chip
  (RFNoC)](https://www.ettus.com/sdr-software/rfnoc/)  APIs that
  support block-level programming of FPGA IP blocks on
  some of the newer USRP radios, such as X3xx and N3/4xx
* **We will use the basic C++ UHD APIs throughout the course**

## [GNU Radio](https://www.gnuradio.org/)
* A software platform for users to build SDR applications 
* Construct *signal flow graphs* that link source, processing, and sink blocks together to implement SDR applications
* Main components:
    - A [scheduler](http://www.trondeau.com/blog/2013/9/15/explaining-the-gnu-radio-scheduler.html)
      that controls and implements signal and data flows among various
      blocks along signal flowgraphs
    - Driver APIs for most commercially available SDR hardwares 
	- A very large collection of pre-defined source, sink, and processing blocks
* Three levels of programming available:
    * [GNU Radio Companion](https://wiki.gnuradio.org/index.php/Guided_Tutorial_GRC): GUI signal flow graph IDE
    * [Python](https://wiki.gnuradio.org/index.php/Embedded_Python_Block): Build signal flowgraphs and other GUIs
    *
      [ C++](https://wiki.gnuradio.org/index.php/Guided_Tutorial_GNU_Radio_in_C%2B%2B): Build source, sink, and processing blocks
* **Although we will NOT use GNU Radio in this class**, I
  suggest everyone learn it because GNU Radio is a great tool for
  developing SDR applications.
  ```{admonition} Why not use GNU Radio as our teaching platform?
  :class: attention
     - We will study some physical-layer (PHY) communication and signal
	 processing techniques/algorithms that are used to implement
	 SDRs. We will definitely learn better by implementing them from
	 scratch.
	 - I believe learning the software plumbings that build SDRs is
       also very important. GNU Radio is great but it hides most of
       the plumbing stuff.
	 - Eliminate the significant overhead of learning how to build
       processing blocks in GNU Radio and work with the GNU Radio
       scheduler.
  ```
	
## References and further reading
```{bibliography}
:style: unsrt
:filter: key % "reed2002" or  key % "UHD" or key % "GNURadio"
```
