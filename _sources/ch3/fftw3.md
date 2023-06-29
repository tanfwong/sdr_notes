# FFTW3 Library 

* The [FFTW3](http://fftw.org/) library is a C/C++ library for
  computing DFT (and some other related transforms) in one or more
  dimensions, of arbitrary input size, and of both real- and
  complex-valued data. The FFTW3 library has been highly optimized to
  run with different levels of precision and under a number of CPU
  architectures. We will employ in most cases the multi-threaded,
  single-precision, 1-d FFT library functions.

* The nice thing about FFTW3 is that a detailed
  manual {cite}`frigo2020` is available. One should
  consult the manual to check out the many different settings,
  configurations, and usages of the library. Here, we briefly summarize
  the steps to use FFTW3 to do a single-precision, 1-d FFT calculation:
  1. Allocate memory to hold the input and output arrays
     ```c++
     fftwf_complex* in = (fftwf_complex*) fftwf_malloc(sizeof(fftw_complex) * N);
     fftwf_complex* out = (fftwf_complex*) fftwf_malloc(sizeof(fftw_complex) * N);
     ```
     The method `fftwf_malloc` should be used to work best with SIMD
     instructions. The type `fftwf_complex` is the compatible with
     `std::complex<float>`.
  1. Instantiate a plan for the FFT using library methods such as `fftwf_plan_dft_1d`:
      ```c++
      fftwf_plan p = fftwf_plan_dft_1d(N, in, out, FFTW_FORWARD, FFTW_ESTIMATE);
      ```
      where `N` specifies the size of the FFT, `FFTW_FORWARD` is a flag
      specifying forward FFT should be calculated, and `FFTW_ESTIMATE` is
      another flag that tells FFTW3 the strategy that it should use to
      speed up the FFT implementation.
  1. Enter the FFT input data to the input array `in`, calculate the
      FFT by executing the FFT plan:
      ```c++
      fftwf_execute(p);
      ```
      and the FFT output will be stored in the array `out` after
      execution.
  1. After all FFT calculations are done, clean up by destroying the
      plan and freeing up the arrays:
      ```c++
      fftwf_destroy_plan(p);
      fftwf_free(in); 
      fftwf_free(out);
      ```
* Let $N$ be the FFT size and $x[n]$ and $X_k$ be the inputs of the
   forward (`FFTW_FORWARD`) and inverse (`FFTW_BACKWARD`) FFT
   plans above, respectively. Executing the forward plan calculates
   \begin{equation*}
     \sum_{n=0}^{N-1} x_n e^{-j \frac{2\pi n k}{N}},
   \end{equation*}
   while executing the inverse plan calculates
   \begin{equation*}
       \sum_{k=0}^{N-1} X_k e^{j \frac{2\pi n k}{N}}.
   \end{equation*}
    Note that one must remember to divide the output of the
   inverse FFT calculation obtained from executing the inverse plan by 
   $N$ in order to get the "standard" inverse FFT result.
* While FFTW3 allows any FFT size, I would recommend sticking to powers of 2 for
   most efficient calculations. 
* In addition to single-precision arithmetic with the `float` type,
  FFTW3 also supports doing calculation with the 
  `double` and `long double` types. To use `double` and `long double`,
  simply replace the prefix `fftwf` in each method or type name by
  `fftw` and `fftwl`, respectively.
* The method `fftwf_plan_dft_1d` above is one of the many *basic
  interface* that FFTW3 provides. A basic interface computes a single
  transform of contiguous data. See {cite}`frigo2020` for other basic
  interfaces provided. FFTW3 also provides two more levels of
  interfaces --- *advanced* and *guru*. An advanced interface computes
  multiple transforms of over contiguous or strided data. A
  guru interface supports the most general data layouts,
  multiplicities, and strides. 
* For later purposes, we often need to perform multiple FFT
  calculations on a batch of samples. To avoid overheads incurred in
  calling a basic interfacing method multiple times, it is beneficial
  to use advanced or guru interface instead. For ease of later use and
  as examples to show the use of the advanced and guru interface, I
  have implemented the class `fft` which serves as a simple wrapper to
  use an advanced interface for doing multi-threaded,
  single-precision, 1-d FFT calculations on a batch of samples in the
  header file [fft.hpp](code:fft). There are two constructors
  implemented:
  - One uses the advanced interface to do multiple FFTs on a long 1-d 
    array of samples.
  - The other uses the guru interface to do multiple FFTs along the rows of a
    2-d array of samples. 
  
  Steps 1-2 and 4 above are implemented in the
  constructors and destructor of `fft` whereas the FFT calculations on
  the whole batch can be executed by calling the class method
  `fft::calculate()`. The "divide-by-$N$" normalization mentioned
  above is included in `fft::calculate()` for inverse FFT.
* To use the wrapper:
  1. Instantiate the `fft` object:
      ```c++
      // To perform nblocks forward fftsize-point FFTs on a 1-d
      // array of samples use the following constructor:
      fft fwd_fft(nthreads, fftsize, nblocks, fftsize, fftsize, false);
      // To perform nblocks inverse fftsize-point FFTs along each 
      // of the D rows of a 2-d array of samples use the following constructor:
      fft inv_fft(nthreads, fftsize, nblocks, fftsize, fftsize, D, true);
      // where both use nthreads threads and move forward
      // fftsize steps to get to the next input and output FFT block
      ```
  2. Get pointer to the input and output arrays:
      ```c++
      std::complex<float>* in = fwd_fft.get_in();
      std::complex<float>* out = fwd_fft.get_out();
      ```
  3. Perform FFT calculations:
      ```c++
      // Load samples into the array pointed to by in
      fwd_fft.calculate();
      // Output stored in the array pointed to by out
      ```
* Refer to [fft.hpp](code:fft) and the example code
  [`test_fft.cpp`](code:test_fft) for detailed information about how
  to use the wrapper.
