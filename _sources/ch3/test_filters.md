(code:test_filters)=
# `test_filters.cpp`

## Source code
```{code-block}  c++
:lineno-start: 1
:emphasize-lines: 0
// University of Florida EEL6528
// Tan F. Wong
// Feb 2, 2021

#include <uhd/utils/safe_main.hpp>
#include <uhd/utils/thread.hpp>
#include <boost/program_options.hpp>
#include <csignal>
#include <chrono>
#include <thread>
#include "filters.hpp"

namespace po = boost::program_options;
namespace clk = std::chrono;

bool stop_signal_called = false;
void sig_int_handler(int){
    stop_signal_called = true;
}

int UHD_SAFE_MAIN(int argc, char *argv[]){

    //variables to be set by po
    int U, D, nthreads, in_len, h_len;
    float freq;
 
    //setup the program options
    po::options_description desc("Allowed options");
    desc.add_options()
        ("help", "help message")
        ("freq,f", po::value<float>(&freq)->default_value(1.0/32), "Use a low-freq sinusoid as input instead of random input")
        ("D,D", po::value<int>(&D)->default_value(1), "Down-sampling factor")
        ("U,U", po::value<int>(&U)->default_value(1), "Up-sampling factor")
        ("in_len,i", po::value<int>(&in_len)->default_value(1000000), "input signal length")
        ("h_len,h", po::value<int>(&h_len)->default_value(64), "Single-rate filter length (multi-rate/polyphase filter length = 4*U*h_len, where h_len = a power of 2)")
        ("nthreads,t", po::value<int>(&nthreads)->default_value(1), "Number of FFT threads to use")
        ;
 
    po::variables_map vm;
    po::store(po::parse_command_line(argc, argv, desc), vm);
    po::notify(vm);
 
    //print the help message
    if (vm.count("help")){
        std::cout << desc << std::endl;
        return EXIT_SUCCESS;
    }

    bool sine = false;
    if (vm.count("sine")) sine = true;
  
    std::signal(SIGINT, &sig_int_handler);

    // Define filter1's impulse response
    int h1_len = h_len;
    std::complex<float> h1[h1_len];
    int delay1 = 2;
    h1[delay1] = 1.0;

    // Filter2 and 3 are the same lowpass filter 
    // This filter design doesn't work too well when D is not a power of 2
    int maxUD = (U>D)?U:D;
    int h2_len = h1_len*U;
    std::complex<float> h2[h2_len];
    fft invfft(1, h2_len, 1, h2_len, h2_len, true);
    std::complex<float>* H2 = invfft.get_in();
    H2[0] = 1.0; 
    for (int i=1; i<h2_len/2/maxUD; i++) {
        H2[i] = H2[0];
        H2[h2_len-i] = H2[0];
    }
    for (int i=h2_len/2/maxUD; i<=h2_len-h2_len/2/maxUD; i++) {
        H2[i] = 0.0;
    }
    invfft.calculate();
    std::copy(invfft.get_out(),invfft.get_out()+h2_len/2, h2+h2_len/2);
    std::copy(invfft.get_out()+h2_len/2,invfft.get_out()+h2_len, h2);
    std::complex<float> h3[h2_len];
    for (int i=0; i<h2_len; i++) {
        h3[i] = h2[i]*float(D)*float(U);
    }
    int delay23 = h2_len/U;
    // Additional delay of 2*D caused by polyphase implementation in
    // filters 4 and 5
    int delay45 = h2_len/U + 2*D; 

    // Construct single-rate OverlapSave filter object
    FilterOverlapSave filter1(in_len, h1_len, h1, nthreads);

    // Construct multi-rate OverlapAdd filter objects
    FilterOverlapSave filter2(U, D, in_len, h2_len, h2, nthreads);
    FilterOverlapSave filter3(D, U, filter2.out_len(), h2_len, h3, nthreads);

    // Construct polyphase OverlapSave filter objects
    FilterPolyphase filter4(U, D, in_len, h2_len, h2, nthreads);
    FilterPolyphase filter5(D, U, filter4.out_len(), h2_len, h3, nthreads);

    // Construct in and out arrays
    int out1_len = filter1.out_len(); // Output length for filter1
    int out2_len = filter2.out_len(); // Output length for filter2
    int out3_len = filter3.out_len(); // Output length for filter3
    int out4_len = filter4.out_len(); // Output length for filter4
    int out5_len = filter5.out_len(); // Output length for filter5
    std::complex<float>* in = new std::complex<float>[in_len]();
    std::complex<float>* out1 = new std::complex<float>[out1_len]();
    std::complex<float>* out2 = new std::complex<float>[out2_len]();
    std::complex<float>* out3 = new std::complex<float>[out3_len]();
    std::complex<float>* out4 = new std::complex<float>[out4_len]();
    std::complex<float>* out5 = new std::complex<float>[out5_len]();

    // Initial timing statistics
    double avg_filter1_time = 0.0;
    double avg_filter2_time = 0.0;
    double avg_filter3_time = 0.0;
    double avg_filter4_time = 0.0;
    double avg_filter5_time = 0.0;
    unsigned count = 0;
    unsigned long icont = 0;
    while (not stop_signal_called) {
        for (int i=0; i<in_len; i++) {
            // complex-valued sinusoid input 
            in[i] = std::complex<float>(cos(2*M_PI*(i+icont)*freq), sin(2*M_PI*(i+icont)*freq));
        }
        
        // Test single-rate overlap-save algorithm using filter1
        clk::steady_clock::time_point start_filter1 = clk::steady_clock::now();
        bool reset = (icont==0);
        filter1.set_head(reset); // clear head of input to filter1
        filter1.filter(in, out1); // Do filtering
        clk::steady_clock::time_point done_filter1 = clk::steady_clock::now();
        clk::microseconds filter1_time = clk::duration_cast<clk::microseconds>(done_filter1-start_filter1);
        // Check output
        float diff = 0.0;
        for (int i=0; i<out1_len-delay1; i++) {
            diff += std::norm(in[i]-out1[i+delay1]);
        }
        printf("\nSingle-rate overlap-save squared error per input sample = %1.4e\n", diff/in_len);

        // Test multi-rate direct overlap-save algorithm using filter2 and then filter3
        clk::steady_clock::time_point start_filter2 = clk::steady_clock::now();
        filter2.set_head(reset);
        filter2.filter(in, out2);
        clk::steady_clock::time_point done_filter2 = clk::steady_clock::now();
        clk::microseconds filter2_time = clk::duration_cast<clk::microseconds>(done_filter2-start_filter2);
        clk::steady_clock::time_point start_filter3 = clk::steady_clock::now();
        filter3.set_head(reset);
        filter3.filter(out2, out3);
        clk::steady_clock::time_point done_filter3 = clk::steady_clock::now();
        clk::microseconds filter3_time = clk::duration_cast<clk::microseconds>(done_filter3-start_filter3);
        // Check output
        diff = 0.0;
        int nstart = (icont==0) ? delay23 : 0;
        for (int i=nstart; i<out3_len-delay23; i++) {
            diff += std::norm(in[i]-out3[i+delay23]);
        }
        printf("Multi-rate overlap-save squared error per input sample = %1.4e\n", diff/(out3_len-nstart-delay23));

        // Test polyphase overlap-add algorithm using filter4 and then filter5
        clk::steady_clock::time_point start_filter4 = clk::steady_clock::now();
        filter4.set_head(reset);
        filter4.filter(in, out4);
        clk::steady_clock::time_point done_filter4 = clk::steady_clock::now();
        clk::microseconds filter4_time = clk::duration_cast<clk::microseconds>(done_filter4-start_filter4);
        clk::steady_clock::time_point start_filter5 = clk::steady_clock::now();
        filter5.set_head(reset);
        filter5.filter(out4, out5);
        clk::steady_clock::time_point done_filter5 = clk::steady_clock::now();
        clk::microseconds filter5_time = clk::duration_cast<clk::microseconds>(done_filter5-start_filter5);

        // Check output
        diff = 0.0;
        nstart = (icont==0) ? delay45 : 0;
        for (int i=nstart; i<out5_len-delay45; i++) {
            diff += std::norm(in[i]-out5[i+delay45]);
        }
        printf("Polyphase overlap-save squared error per input sample = %1.4e\n", diff/(out5_len-nstart-delay45));
        // Print calculation times of filters
        printf("Single-rate overlap-save filter1 processing time = %ld us\n", filter1_time.count());
        printf("Multi-rate overlap-save filter2 processing time = %ld us\n", filter2_time.count());
        printf("Multi-rate overlap-save filter3 processing time = %ld us\n", filter3_time.count());
        printf("Polyphase overlap-save filter4 processing time = %ld us\n", filter4_time.count());
        printf("Polyphase overlap-save filter5 processing time = %ld us\n", filter5_time.count());

        // Accummulate timing stats
        count++;
        avg_filter1_time += filter1_time.count();
        avg_filter2_time += filter2_time.count();
        avg_filter3_time += filter3_time.count();
        avg_filter4_time += filter4_time.count();
        avg_filter5_time += filter5_time.count();

        icont += in_len;
    } 

    printf("\n\nSingle-rate overlap-save filter1 average processing time = %1.4e us\n", avg_filter1_time/count);
    printf("Multi-rate overlap-save filter2 average processing time = %1.4e us\n", avg_filter2_time/count);
    printf("Multi-rate overlap-save filter3 average processing time = %1.4e us\n", avg_filter3_time/count);
    printf("Polyphase overlap-save filter4 average processing time = %1.4e us\n", avg_filter4_time/count);
    printf("Polyphase overlap-save filter5 average processing time = %1.4e us\n", avg_filter5_time/count);
    
    delete in;
    delete out1;
    delete out2;
    delete out3;
    delete out4;
    delete out5;
    fft::cleanup();
    return EXIT_SUCCESS;
}
```
* `filter1` implements single-rate frequency-domain filtering using
the overlap-save algorithm. The impulse response of `filter1` is
chosen to delay the input samples by 2 so that we can easily verify if
the single-rate filter implementation in `FilterOverlapSave` is
correct or not.
* `filter2` and `filter3` implement direct multi-rate
frequency-domain filtering using the overlap-save algorithm. The
interpolation and decimation rates of `filter2` and `filter3` are
reversed. The filter impulse responses are chosen for interpolation and
anti-aliasing purposes, resepctively. The filter design only works well when the
single-rate filter length parameter `h_len` is set to some power of 2.
Because `filter3` aims to reverse the action of `filter2`, we can
easily verify if the multi-rate filter implementation in
`FilterOverlapSave` is correct or not.
* `filter4` and `filter5` are polyphase impoementations of `filter2`
  and `filter3`, respectively. The only difference is that our
  polyphase implementation introduces an additional delay of $U$
  samples, where $U$ is the up-sampling factor. Hence, `filter4` adds
  a delay of $U$ samples at its output. Recall that `filter5` flips
  the up- and down-sampling factors of `filter4`. Hence, the
  $U$-sample delay of `filter4` is converted to $D$ samples by
  `filter5`, which adds another delay of $D$ samples to its own
  output. Thus, the overall delay at the cascade of the two filters
  are $2D$ samples at `filter5`'s output.

## Build
* Example Makefile for building `test_fft` and `test_filters` in
  Unix-based system: [`Makefile`](code:Makefile_fft_filters)

```{admonition} Experiment
:class: hint
1. Compile and run the test code. 
1. Try different choices of $U$ and $D$ to see if the advantages in
  computational complexity predicted in the
  [polyphase filtering discussion](sec:polyphase) match up with the
  implementation in [`filters.cpp`](code:filters_cpp). 
1. Try to increase the number of threads used in the FFT calculations
  to see if and whichfiltering operations can be sped up.
1. Implement your own direct multi-rate and polyphase filters by doing
  convolutions in time-domain. Compare the filtering speeds of your
  time-domain implementations with the respective frequency-domain
  implementations in [`filters.cpp`](code:filters_cpp). 
```
