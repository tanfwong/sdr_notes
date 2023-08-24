(sec:UHD)=
# UHD
* UHD is the device driver for USRP radios.
* It provides C++ APIs to set up, query, and control a USRP radio and
  to transfer signal samples between the USRP radio and a host
  computer.
* The version of UHD that we will use in class (installed on `lab-1`
  and `lab-2`) is [UHD 4.4.0](https://files.ettus.com/manual/).


## UHD_SAFE_MAIN macro and Termination trick
* [`UHD_SAFE_MAIN`](code:uhd_safe_main) is a macro for `main()` that
  catches and prints out an exception, in case it is thrown, to
  `stderr`. It is not necessary, but good to use the macro. 
* In many cases, we want our `main()` and other functions to keep running until we decide otherwise. Thus we need a way to terminate the program. The simplest way to do so is to issue a CTRL-C interrupt. See below for a sample code to catch the CTRL-C interrupt and terminate `main()`.
* <u>Example</u>: 
  ```c++
  #include <uhd/utils/safe_main.hpp>
  #include <csignal>

  // Interrupt signal handler
  static bool stop_signal_called = false;
  void sig_int_handler(int) {
      stop_signal_called = true;
  }

  int UHD_SAFE_MAIN(int argc, char *argv[]) {
      ...
	  // Set the signal handler
      std::signal(SIGINT, &sig_int_handler);
      ...
      // Keep running until CTRL-C issued
      while(not stop_signal_called){
          ...
      }
      return EXIT_SUCCESS;
  }
  ```


## Boost.Program_options Library
* [Boost](http://www.boost.org/) is a huge collection of C++ libraries
  that help us do many common application tasks. We will mainly use
  the Boost libraries for option parsing, printing, and sometimes
  threading. The version of Boost installed installed on
  `lab-1` and `lab-2` is [Boost 1.74.0](https://www.boost.org/doc/libs/1_74_0/).
* In particular,
  [Boost.Program_options](https://www.boost.org/doc/libs/1_74_0/doc/html/program_options.html)
  library will come in handy when parsing program options. 
* <u>Example</u>: [`age.cpp`](code:age)

## Set up, control, and query USRP
1. Use the `uhd::usrp::multi_usrp` class method
   `uhd::usrp::multi_usrp::make()` to construct a USRP object.
2. Use other `uhd::usrp::multi_usrp` class APIs to set and query various USRP parameters. 
* A full list of `uhd::usrp::multi_usrp` class APIs can be found
  [here](https://files.ettus.com/manual/classuhd_1_1usrp_1_1multi__usrp.html).
* <u>Example</u>: [`txrx_loopback_to_file.cpp`](code:txrx_setup)

## USRP Streams
* To transmit/receive using an N210 USRP, signal samples need to be
  transferred to/from the USRP. This is accomplished 
  via the Gigabit Ethernet connection between the host computer and
  the N210 USRP.
* Signal samples are encapsulated in Ethernet packets that are
  transferred between the host computer and USRP. This is referred to
  as **streaming**.
* To do streaming, we need to first construct a **stream** object that
  facilitates streaming between the host computer and USRP. There are
  two kinds of stream objects:
  - A **TX stream** allows the host computer to transmit samples to the
      USRP, and is constructed using the
      `uhd::usrp::multi_usrp::get_tx_stream()` method, which returns a
      smart pointer to a `uhd::tx_streamer` class object.
  - A **RX stream** allows the host computer to receive samples from the
      USRP, and is constructed using the
      `uhd::usrp::multi_usrp::get_rx_stream()` method, which returns a
      smart pointer to a `uhd::rx_streamer` class object.
* Both stream construction methods require an input argument of type
    `uhd::stream_args_t(const std::string& cpu, const std::string&
    otw)`, where `cpu` and `otw` represent the different data types of
    signal samples at the two ends of a stream:
   - **`cpu` specifies the host data type:** Data type of the samples used on the
     host for processing, possible choices are  `fc64`, `fc32`, `sc16`, and `sc8`.
   - **`otw` specifies USRP over-the-wire data type:** Data type of
     the samples sent through Ethernet, possible choices are `sc16`
     and `sc8`
   - `fc64` = `std::complex<double>`, `fc32` = `std::complex<float>`,
     `sc16` = `std::complex<int16_t>`, and `sc8` = `std::complex<int8_t>`.
   - See [`stream.hpp`](code:stream) in the UHD source package for
     detailed specifications of the sample data types above, as well
     as other additional fields in `uhd::stream_args_t`.
* Data-type conversion is automatically carried out by the stream
  objects.
* <u>Example</u>: [`txrx_loopback_to_file.cpp`](code:txrx_stream)

## TX: Send signal samples to USRP
* After a TX stream object is constructed, we may use the method
  `uhd::tx_streamer::send()` to transfer signal samples from the host
  to the USRP. See code examples below and [`stream.hpp`](code:stream)
  to see all input arguments of `uhd::tx_streamer::send()`. The method
  `uhd::tx_streamer::send()` is a blocking call and will not return
  until the specified number of samples have been read out of from the
  buffer, unless a timeout occurs.
* Note that the samples are encapsulated into Ethernet packets for the
  stream transfer. Each packet can only carry a certain maximum number
  of samples, which is determined by the Ethernet hardware and the
  amount of buffering in the USRP and the host. This maximum number
  can be obtained using the `uhd::tx_streamer::get_max_num_samps()`
  method.
* If the total number of samples is greater than this maximum number,
  then we have to chop up the samples into multiple Ethernet packets
  and send them to the USRP in a *burst*. This may be done
  manually. However, `uhd::tx_streamer::send()` will automatically do
  that for us. Specifically, it will fragment the samples across
  several packets.  I recommend letting `uhd::tx_streamer::send()`
  handle that.
* In either case, we need to tell the USRP when the burst starts and
  ends. This is done by setting `start_of_burst` and `end_of_burst`
  flags in a metadata structure of type `uhd::tx_metadata_t`
  associated with each call to `uhd::tx_streamer::send()`. See code
  examples below for different ways to set and use these parameters.
  In the case of auto-fragmentation, `uhd::tx_streamer::send()` will
  respect the burst flags when fragmenting to ensure that
  `start_of_burst` can only be set on the first fragment and that
  `end_of_burst` can only be set on the final fragment.
* Also we can ask UHD to stream the samples immediately by setting the
  value of the `has_time_spec` field to `false` (which is the default
  value) in the metadata. Or we may ask UHD to not start streaming
  before a certain time. See the
  [`txrx_loopback_to_file.cpp`](code:txrx_TX) example below for how to
  do that.
* After the USRP receives the samples, it takes over the actual
  digital up conversion, D/A conversion, carrier mixing, filtering,
  and transmission processes.
* One needs to be aware of the fact that there is a certain latency
  from sending the first packet of samples using
  `uhd::tx_streamer::send()` to the actual physical transmission of
  the signal. This latency may vary with the host computer's load and
  the traffic load over the Ethernet connection.
* The USRP consumes, and hence expects, samples coming in at a
constant rate. An underflow occurs when the host does not produce (or
send) samples fast enough. When UHD detects the underflow, it prints a
"U" to `stdout`, and pushes a message packet into the async message
stream back to the host.
* <u>Example of transmitting a burst of samples:</u> [`tx_samples_from_file.cpp`](code:tx_samples)
* <u>Example of continuous transmission:</u> [`txrx_loopback_to_file.cpp`](code:txrx_TX)

## RX: Grab signal samples from USRP
* After a RX stream object is constructed, we may use the method
  `uhd::rx_streamer::recv()` to transfer signal samples from the USRP
  to the host.
* Note that the samples are encapsulated into Ethernet packets for the
  stream transfer.  Each packet can only carry a certain maximum
  number of samples, which is determined by the Ethernet hardware and
  the amount of buffering in the USRP and the host. This maximum
  number can be obtained using the
  `uhd::rx_streamer::get_max_num_samps()` method. By default, UHD
  sends this maximum number of samples.  One may specify the number of
  samples to be transferred per packet by setting the `spp` field in
  the `uhd::stream_args_t::args` structure when constructing the RX
  stream. See [`stream.hpp`](code:stream) for details.
* The `uhd::rx_streamer::recv()` method is a blocking call and will
  not return until the number of samples specified have been written
  into the buffer, unless a timeout occurs. Refer to the code example
  below and [`stream.hpp`](code:stream) for how to specify the number
  of samples to receive and to see all input arguments of
  `uhd::rx_streamer::recv()`. 
* On the other hand, if the buffer has insufficient space (the
  specified number of samples) to hold all samples that were received
  in a single packet from the USRP, then the buffer will be completely
  filled and `uhd::rx_streamer::recv()` will hold a pointer into the
  remaining portion of the packet. Subsequent calls will load from the
  remainder of the packet, and will properly set the `start_of_burst`,
  `end_of_burst`, and `fragment_offset` fields in the returned
  `uhd::rx_metadata_t` structure to show that this is a fragment (see
  [](https://files.ettus.com/manual_archive/v3.15.0.0/html/structuhd_1_1rx__metadata__t.html)
  for details). The next call to receive, after the remainder becomes
  exhausted, will get a new packet from the USRP as usual.
* There are two different modes of streaming samples from the USRP,
  namely **continuous** and **fixed-number-of-samples**. Operations of
  both modes are controlled by issuing a **stream command** to the
  USRP using the `uhd::rx_streamer::issue_stream_cmd()` method. See
  the code example below for how to use the method.
* A stream command is of the `uhd::stream_cmd_t` type, and the
 different modes of streaming can be specified by setting the
 `stream_mode` field or by using the constructor of
 `uhd::stream_cmd_t`. Possible options for `stream_mode` include:
  - `STREAM_MODE_START_CONTINUOUS` tells the USRP to stream samples
 indefinitely
  - `STREAM_MODE_STOP_CONTINUOUS` tells the USRP to end continuous
 streaming
  - `STREAM_MODE_NUM_SAMPS_AND_DONE` tells the USRP to stream
  `num_samps` samples and to not expect a future stream command for
  contiguous samples
  - `STREAM_MODE_NUM_SAMPS_AND_MORE` tells the USRP to stream
 `num_samps` samples and to expect a future stream command for
 contiguous samples.
* We will mostly do continuous streaming later. Once the stream
  command is issued, the USRP will automatically take over the actual
  receiving, frequency mixing, A/D conversion, filtering, digital down
  sampling, buffering, and transfer of the signal samples to the
  host. 
* One may specify the number of requested samples for the
  fixed-number-of-samples mode of streaming by setting the
  `num_samps` field in the stream command. Also one can
  tell the USRP to start streaming immediately by setting the
  `stream_now` field in the stream command to `true` or at a later
  specific time by specifying the `time_spec` field. See the code
  example below for how to set the `time_spec` field.
* A RX metadata structure `uhd::rx_metadata_t` is available to describe
  information about the streaming process and the sample packet. Most
  useful to us will be the `error_code` field of the RX metadata. The
  possible flags of the field are:
  - `ERROR_CODE_NONE`: no error associated with this metadata
  - `ERROR_CODE_TIMEOUT`: no packet received, implementation timed-out
  - `ERROR_CODE_LATE_COMMAND`: a stream command was issued in the past
  - `ERROR_CODE_BROKEN_CHAIN`: expected another stream command
  - `ERROR_CODE_OVERFLOW`: an overflow has occurred
  - `ERROR_CODE_ALIGNMENT`: ,ulti-channel alignment failed
  - `ERROR_CODE_BAD_PACKET`: the packet could not be parsed
* One may use the `uhd::rx_metadata_t ::strerror()` method to print out the error.
* The most common error is an overflow which occurs when the host does
  not consume data provided by the USRP fast enough. When UHD detects
  the overflow, it prints an "O" or "D" to `stdout`, and pushes a
  message packet into the RX stream.
* <u>Example</u>: [txrx_loopback_to_file.cpp (RX)](code:txrx_RX)

