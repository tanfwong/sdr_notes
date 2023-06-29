(code:fft)=
# `fft.hpp`

## Source code
```{code-block}  c++
:lineno-start: 1
:emphasize-lines: 0
// University of Florida EEL6528
// Tan F. Wong
// Feb 2, 2021

#ifndef FFT_H
#define FFT_H
#include <complex>
#include <fftw3.h>
//#include <thread>
#ifdef USE_VOLK
#include <volk/volk.h>
#endif

// 1-d, single-precision FFT class
// All samples in the arrays in, out are contiguous
// out_nsteps >= fftsize
// Use multi-threading in FFTW3
class fft{
  private:
    fftwf_plan plan; // FFT plan
    int nthreads;   // Number of threads
    int fftsize;    // FFT size
    int nblocks;    // Number of FFT blocks to calculate along a row
    int in_nsteps;  // How many samples to next block in array in
    int out_nsteps; // How many samples to next block in array out
    int U_D;          // # of rows in 2-d array = Up/Down-sampling factor
    bool inverse;   // FFT = false, IFFT = true
    std::complex<float>* in;      // Pointer to input array
    std::complex<float>* out;     // Pointer to output array
    std::complex<float>* ptr_in;  // Pointer to hold if want to allocate memory for input array
    float one_over_fftsize; // 1/fftsize
    std::complex<float> one_over_fftsize_c; // 1/fftsize complex value
 
  public:
    // Constructor for calculating multiple FFTs along a 1-d array
    fft(int nt, int n, int nblks, int in_ns, int out_ns, bool inv=false, std::complex<float>* const pin=NULL) {
        // Set class variables
        nthreads = nt;
        fftsize = n;
        one_over_fftsize = 1.0f/fftsize;
        one_over_fftsize_c =  std::complex<float>(one_over_fftsize, 0.0f);
        nblocks = nblks;
        in_nsteps = in_ns;
        out_nsteps = std::max(n, out_ns);
        U_D = 1;
        inverse = inv;
        ptr_in = pin;
        // Allocate input array
        if (ptr_in==NULL) 
            // a null ptr tells fft to allocate memory to input array
            in = (std::complex<float>*) fftwf_malloc(nblocks*std::max(in_nsteps,fftsize)*sizeof(std::complex<float>));
        else
            // a non-null ptr_in tells fft not to allocate memory to input array
            in = ptr_in;
        // Allocate output array
        out = (std::complex<float>*) fftwf_malloc(nblocks*out_nsteps*sizeof(std::complex<float>));
        // Initialize FFTW3 multi-threaded implementation
        fftwf_init_threads();
        fftwf_plan_with_nthreads(nthreads);
        // InstantiateForward FFT plan usinf FFTW3's advanced interface 
        plan = fftwf_plan_many_dft(1, &fftsize, nblocks, 
                                  reinterpret_cast<fftwf_complex *>(in), 
                                  NULL, 1, in_nsteps, 
                                  reinterpret_cast<fftwf_complex *>(out), 
                                  NULL, 1, out_nsteps, 
                                  inverse ? FFTW_BACKWARD : FFTW_FORWARD, 
                                  FFTW_MEASURE);
    }
    // Constructor calculating multiple FFTs along each row of a 2-d array
    // for use in polyphase filtering
    // Assume input array arranged with signals in uord rows 
    // in_ns and out_ns correspond to the number of samples along each row
    // nblks also indicates the number of FFT block per row
    fft(int nt, int n, int nblks, int in_ns, int out_ns, int uord, bool inv=false, std::complex<float>* const pin=NULL) {
        // Set class variables
        nthreads = nt;
        U_D = uord;
        fftsize = n;
        one_over_fftsize = 1.0f/fftsize;
        one_over_fftsize_c =  std::complex<float>(one_over_fftsize, 0.0f);
        nblocks = nblks;
        in_nsteps = in_ns;
        out_nsteps = std::max(n, out_ns);
        inverse = inv;
        ptr_in = pin;
        // Allocate input array
        if (ptr_in==NULL) 
            // a null ptr tells fft to allocate memory to input array
            in = (std::complex<float>*) fftwf_malloc(nblocks*U_D*std::max(in_nsteps,fftsize)*sizeof(std::complex<float>));
        else
            // a non-null ptr_in tells fft not to allocate memory to input array
            in = ptr_in;
        // Allocate output array
        out = (std::complex<float>*) fftwf_malloc(nblocks*U_D*out_nsteps*sizeof(std::complex<float>));
        // Calculate parameters for multi-threaded implementation
        // Initialize FFTW3 multi-threaded implementation
        fftwf_init_threads();
        fftwf_plan_with_nthreads(nthreads);
        // InstantiateForward FFT plan using FFTW3's guru interface
        // Set up fft and howmany dimensions
        fftwf_iodim fft_dim = {fftsize, 1, 1};
        fftwf_iodim howmany_dim[2]; // 3d arrays for input and output
        // This is to match the 1d array of input
        howmany_dim[1] = {nblocks, in_nsteps, out_nsteps}; 
        howmany_dim[0] = {U_D, nblocks*fftsize, nblocks*fftsize};
        plan = fftwf_plan_guru_dft(1, &fft_dim, 2, howmany_dim,
                                  reinterpret_cast<fftwf_complex *>(in), 
                                  reinterpret_cast<fftwf_complex *>(out), 
                                  inverse ? FFTW_BACKWARD : FFTW_FORWARD, 
                                  FFTW_MEASURE);
    }
    // Destructor
    ~fft(void) {
        if (ptr_in==NULL) fftwf_free(in);
        fftwf_free(out);
        fftwf_destroy_plan(plan);
    }
    // Parameter getters
    int get_fftsize(void) {
        return fftsize;
    }
    int get_nblocks(void) {
        return nblocks;
    }
    int get_in_nsteps() {
        return in_nsteps;
    }
    int get_out_nsteps(void) {
        return out_nsteps;
    }
    std::complex<float>* get_in(void) {
        return in;
    }
    std::complex<float>* get_out(void) {
        return out;
    }
    bool is_inverse(void) {
        return inverse;
    }
    // Clean up
    static void cleanup(void) {
        fftwf_cleanup();
        fftwf_cleanup_threads();
    }
    // Calculate FFT/IFFT
    void calculate(void) {
        // Call FFTW3 to Calculate FFT/IFFT
        fftwf_execute(plan);
        // If IFFT, divide by fftsize
        if (inverse) {
            for (int i=0; i<U_D*nblocks; i++) {
                unsigned idx = i*out_nsteps;
#ifdef USE_VOLK
                volk_32fc_s32fc_multiply_32fc(out+idx, out+idx, one_over_fftsize_c, fftsize);
#else
                for (int j=0; j<fftsize; j++) {
                    out[idx++] *= one_over_fftsize;
                }
#endif
            }
        }
    }
};
#endif //FFT_H
```
* The `fft` class defined above uses the FFTW3 advanced interface
  `fftwf_plan_many_dft` to create a plan to calculate multiple
  single-precision, 1-d FFTs along a 1-d array of samples and the guru
  interface `fftwf_plan_guru_dft` to create a plan to calculate
  multiple single-precision, 1-d FFTs along each row of a 2-d array of
  samples, using potentially multiple threads.
* Both `fftwf_plan_many_dft` and `fftwf_plan_guru_dft` are actually
  more flexible that the way they are used here. Please refer to
  {cite}`frigo2020` for more usage information.
* Can define the compile flag `USE_VOLK` to use the 
  [VOLK](https://www.libvolk.org/) library for SIMD-accelerated
  arithmetic kernels such that `volk_32fc_s32fc_multiply_32fc()`.
* The internal multi-threading implementation of FFTW is used. 
   ```{caution}
   1. The internal multi-threading implementation of FFTW seems to
        interfere with setting the thread priority using the IHD
        method `uhd::set_thread_priority_safe()`. In particular,
        `priority` is set to the highest value regardless of what we
        set `priority` to, but `realtime` is not affected. I
        haven't been able to work out the exact reason for that. FFTW3
        uses Pthreads to set the priority of its threads and
        hence probably overwrites the value set by `uhd::set_thread_priority_safe()`.
    1. Setting `realtime` to `true` actually results in slower FFT
        calculations in this case.
   ```

## Usage
* 1-d constructor: 
  ```c++
  fft(int nthreads, int fftsize, int nblocks, int in_nsteps, int out_nsteps, bool inverse=false, std::complex<float>* const in_ptr=NULL) 
  
  ```
* 2-d constructor: 
  ```c++
  fft(int nthreads, int fftsize, int nblocks, int in_nsteps, int out_nsteps, int nrows, bool inverse=false, std::complex<float>* const in_ptr=NULL) 
  ```
  - `nthreads`: number of threads to use
  - `fftsize` : size of FFT/IFFT
  - `nblocks`: number of FFTs/IFFTs to calculate along a row
  - `in_nsteps`: number of samples to step to the next
      block along a row in the input array
  - `out_nsteps`: number of samples to step to the
      next block along a row in the output array
  - `nrows`: number of rows in the 2-d array
  - `inverse`: FFT = `false `, IFFT = `true` (default = `false`)
  - `in_ptr`: pointer to the input array if non-`NULL`, 
     internally allocate memory for the input array if `NULL` 
     (default = `NULL`)
* The restriction that `out_nsteps` must be at least as large as
  `fftsize` is imposed while `in_nsteps` may be smaller than
  `fftsize`.
* The methods `fft::get_in()` and `fft::get_out()` can be used to
  obtain the pointers to the input and output arrays. The elements of
  both arrays should be of type `std::complex<float>`. The numbers of
  elements allocated for the input and output arrays are
  `nblocks`$\cdot \max($`fftsize`$,$ `in_nsteps`$)$ and `nblocks`$\cdot
  \max($`fftsize`$,$ `out_nsteps`$)$, respectively.
* For nice termination, call the method `fft::cleanup()` before your
  `main()` exits.
* For usage examples of the 1-d constructor, see
  [`test_fft.cpp`](code:test_fft).
* For usage examples of the 2-d constructor, see
  [`filters.cpp`](code:filters_cpp).
