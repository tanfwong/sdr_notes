(code:test_fft)=
# `test_fft.cpp`

## Source code
```{code-block}  c++
:lineno-start: 1
:emphasize-lines: 50, 53, 71, 75, 106
// University of Florida EEL6528
// Tan F. Wong
// Jan 18, 2021

#include <uhd/utils/safe_main.hpp>
#include <boost/program_options.hpp>
#include <uhd/utils/thread.hpp>
#include <csignal>
#include <chrono>
#include "fft.hpp"

namespace po = boost::program_options;
namespace clk = std::chrono;

static bool stop_signal_called = false;
void sig_int_handler(int){
    stop_signal_called = true;
}

int UHD_SAFE_MAIN(int argc, char *argv[]){

    // Set main thread priority to highest
    uhd::set_thread_priority_safe(1.0, false);
 
    //variables to be set by po
    int fftsize, nblocks, nthreads;
 
    //setup the program options
    po::options_description desc("Allowed options");
    desc.add_options()
        ("help", "help message")
        ("fftsize,n", po::value<int>(&fftsize)->default_value(1024), "FFT size")
        ("nblocks,b", po::value<int>(&nblocks)->default_value(1000), "Number of FFT blocks to calculate")
        ("nthreads,t", po::value<int>(&nthreads)->default_value(1), "Number of threads to use")
    ;
 
    po::variables_map vm;
    po::store(po::parse_command_line(argc, argv, desc), vm);
    po::notify(vm);
 
    //print the help message
    if (vm.count("help")){
        std::cout << desc << std::endl;
        return EXIT_SUCCESS;
    }

    std::signal(SIGINT, &sig_int_handler);

    // Instantiate forward FFT (1st false) and allocate memory for input array
    fft fwd(nthreads, fftsize, nblocks, fftsize, fftsize, false);
    // Instantiate inverse FFT (1st true)
    // DON'T allocate memory for input array but set it to fwd's output 
    fft inv(nthreads, fftsize, nblocks, fftsize, fftsize, true, fwd.get_out());

    // Set random number generator seed
    srand(time(NULL));
    std::complex<float>* in = fwd.get_in();
    std::complex<float>* out = inv.get_out();

    // Initial timing statistics
    double avg_fft_time = 0.0;
    double avg_ifft_time = 0.0;
    unsigned count = 0;
    while (not stop_signal_called) {
        // Generate samples with random complex-values
        for (int i=0; i<fftsize*nblocks; i++) {
            in[i] = std::complex<float>(float(rand())/float(RAND_MAX), float(rand())/float(RAND_MAX));
        }
        // Execute forward FFT
        clk::steady_clock::time_point start_fft = clk::steady_clock::now();
        fwd.calculate();
        clk::steady_clock::time_point done_fft = clk::steady_clock::now();
        // Execute inverse FFT
        clk::steady_clock::time_point start_ifft = clk::steady_clock::now();
        inv.calculate();
        clk::steady_clock::time_point done_ifft = clk::steady_clock::now();
        // Get the time required to calculate FFT and IFFT
        clk::microseconds fft_time = clk::duration_cast<clk::microseconds>(done_fft-start_fft);
        clk::microseconds ifft_time = clk::duration_cast<clk::microseconds>(done_ifft-start_ifft);
        
        // Check whether doing forward and then inverse FFT gives back the samples
        float diff = 0.0;
        for (int i=0; i<fftsize*nblocks; i++) {
            diff += std::norm(in[i]-out[i]);
        }
        diff /= nblocks*fftsize;
        std::cout << "Squared difference per sample = " << diff << std::endl;
        if (diff <= 1e-10)
            std::cout << "FFT has worked correctly!" << std::endl;
        else
            std::cout << "FFT has failed!\n\n"<< std::endl;
        std::cout << "It took " << fft_time.count() << " us to compute " << nblocks << " blocks of " << fftsize <<"-point FFT\n";
        std::cout << "It took " << ifft_time.count() << " us to compute " << nblocks << " blocks of " << fftsize <<"-point IFFT\n\n";
        // Accummulate timing stats
        count++;
        avg_fft_time += fft_time.count();
        avg_ifft_time += ifft_time.count();
        
        std::this_thread::sleep_for(std::chrono::milliseconds(10)); 
    }

    std::cout << std::endl;
    std::cout << "Average time to compute " << nblocks << " blocks of " << fftsize <<"-point FFT is " << avg_fft_time/count << " us\n" ;
    std::cout << "Average time to compute " << nblocks << " blocks of " << fftsize <<"-point IFFT is " << avg_ifft_time/count << " us\n\n" ;

    fft::cleanup();
    return EXIT_SUCCESS;
}
```
* This test program do `fftsize`-point FFT and then IFFT on each block
  of samples for `nblocks` blocks using the `fft` wrapper class.
* It also estimates the average run times for the FFT and IFFT
  calculations.

## Build
* Need to link the `uhd`, `boost_program_options`, `fftw3f`,
  and `fftw3f_threads` libraries when compiling.
* Can define the compiler flag `USE_VOLK` to use the `volk` library (see [`fft.hpp`](code:fft)).
  ```
  g++ test_fft.cpp -o test_fft -luhd -lboost_program_options -lfftw3f -lfftw3f_threads -DUSE_VOLK -lvolk 
  ```

## Tests
  ```{admonition} Experiment
  :class: hint
  1. Use the default values of `fftsize` and `nblocks`. Run the
      program with increasing numbers of threads, starting from one
      thread, to see how much speed advantage you can obtain.
  1. Redo 1. with different values of `fftsize` and `nblocks` to see
      the effects of the parameters on the speed.
  1. Recompile without defining the `USE_VOLK` flag. Run the program
      again to see how much reduction in speed results.
  ```
