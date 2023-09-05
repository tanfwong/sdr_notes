(code:filters_hpp)=
# `filters.hpp`

## Source code
```{code-block}  c++
:lineno-start: 1
:emphasize-lines: 0
// University of Florida EEL6528
// Tan F. Wong
// Feb 2, 2021

#pragma once

#include "fft.hpp"

// Overlap-save filter class
class FilterOverlapSave {
  public:
    FilterOverlapSave(int x_len, int h_len, std::complex<float>* h, int nt=1); // single-rate constructor
    FilterOverlapSave(int up, int down, int x_len, int h_len, std::complex<float>* h, int nt=1); // multi-rate constructor
    ~FilterOverlapSave(void); // destructor
    int filter(std::complex<float>* in, std::complex<float>* out);  //do filtering
    void set_head(bool reset=false); // Set head of input array for continuously filtering
    int out_len(void); //calculate output length

  private:
    void setup(int x_len, int h_len, std::complex<float>* h, int nt); // utility method to set up FFTs
    int fftsize;
    fft* fwdfft; // forward FFT 
    fft* invfft; // inverse FFT
    int U; // upsampling factor
    int D; // downsampling factor
    int L; // length of filter impulse response
    int M; // length of valid sample block
    int nblocks; // number of blocks to calculate
    int nthreads; // number of threads to use in FFT calculation
    int nx; // length of input sequence
    int Unx; // length of input sequence after interpolation
    std::complex<float>* H; // FFT of impulse response
};

// Polyphase filter class
class FilterPolyphase {
  public:
    FilterPolyphase(int up, int down, int x_len, int h_len, std::complex<float>* h, int nt); // constructor
    ~FilterPolyphase(void); // destructor
    int filter(std::complex<float>* in, std::complex<float>* out);  // do filtering
    void set_head(bool reset=false); // Set head of input array for continuously filtering
    int out_len(void); //calculate output length

  private:
    int fftsize;
    fft* fwdfft; // forward FFT 
    fft* invfft; // inverse FFT
    int U; // upsampling factor
    int D; // downsampling factor
    int L; // length of filter impulse response
    int Lp; // length of polyphase filter impulse responses
    int M; // length of valid sample block
    int nblocks; // number of blocks to calculate on each down-sampled input signal (row)
    int nthreads; // number of threads to use in FFT calculation
    int nx; // length of input sequence
    int nxD; // length of down-sampled signals
    std::complex<float>* F; // FFTs of impulse responses of polyphase filters
};
```
