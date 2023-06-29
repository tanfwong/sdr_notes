(code:filters_cpp)=
# `filters.cpp`

## Implementation
1. Multi-rate direct overlap-save filter class: `FilterOverlapSave`
1. Polyphase overlap-save filter class: `FilterPolyphase`

* <u>Header file</u>: [`filters.hpp`](code:filters_hpp)

## Source code
```{code-block}  c++
:lineno-start: 1
:emphasize-lines: 0
// University of Florida EEL6528
// Tan F. Wong
// Feb 2, 2021

#include "filters.hpp"

// OverlapSave filter class
// Set up FFT objects and filter's frequency response
void FilterOverlapSave::setup(int x_len, int h_len, std::complex<float>* h, int nt)
{
    // Set up sizes for the overlap-save algorithm
    nx = x_len;
    Unx = nx*U;
    if (Unx%D != 0)
        printf("[WARNING] For proper continuous filtering, input_length*U must be divisible by D!\n");
    L = h_len;
    double LL = (L<32) ? 32.0 : double(L); // If L too small, use 32
    // Choose FFT size to be 2^m, where m is smallest power that 2^m >= 8*L
    fftsize = int(pow(2.0, ceil(log2(LL)+3)));
    
    M = fftsize-L+1;
    // Work out the number of FFT blocks needed
    nblocks = int(ceil(double(Unx)/M));
    // set number of threads to use 
    nthreads = nt;
    
    // Set up forward and inverse FFT objects
    fwdfft = new fft(nthreads, fftsize, nblocks, M, fftsize, false);
    invfft = new fft(nthreads, fftsize, nblocks, fftsize, fftsize, true, fwdfft->get_out());

    // Calculate and save FFT of h
    
    fft fft1time(1, fftsize, 1, 0, 0, false);
    std::complex<float>* a = fft1time.get_in();
    for (int i=0; i<L; i++) {
        a[i] = h[i];
    }
    for (int i=L; i<fftsize; i++) {
        a[i] = 0.0;
    }
    fft1time.calculate();
    H = (std::complex<float>*) fftwf_malloc(fftsize*sizeof(std::complex<float>));
    std::copy(fft1time.get_out(), fft1time.get_out()+fftsize, H);

    // Set whole fwd->in to 0
    std::fill(fwdfft->get_in(), fwdfft->get_in()+nblocks*fftsize, 0.0);
}

// Set head for next call in continuous filtering
// For continuous filtering U*(length of input) must be divisible by D
void FilterOverlapSave::set_head(bool reset) {
    std::complex<float>* in = fwdfft->get_in();
    if (reset)
        std::fill(in, in+L-1, 0.0);
    else
        std::copy(in+Unx, in+Unx+L-1, in);
}

// Single-rate FilterOverlapSave class constructor (U=D=1)
FilterOverlapSave::FilterOverlapSave(int x_len, int h_len, std::complex<float>* h, int nthreads)
{
    // Set up upsampling and downsampling rates to 1
    U = 1;
    D = 1;
    // Set up FFTs and filter's freq response
    this->setup(x_len, h_len, h, nthreads);
}

// Multi-rate FilterOverlapSave class constructor
FilterOverlapSave::FilterOverlapSave(int up, int down, int x_len, int h_len, std::complex<float>* h, int nthreads)
{
    // Set up upsampling and downsampling rates to specified values 
    U = up;
    D = down;
    // Set up FFTs and filter's freq response
    this->setup(x_len, h_len, h, nthreads);
}

// Class destructor
FilterOverlapSave::~FilterOverlapSave(void) {
    fftwf_free(H);
    delete fwdfft;
    delete invfft;
}

// Calculate output length
// N.B.: The implementation of the filter method below outputs
// floor(U*len(input)/D) samples. The transient in the beginning is not
// included in the output. This is done so that the filter method 
// may be used to filter a continuous stream of samples.
int FilterOverlapSave::out_len(void) {
    return int(floor(double(Unx)/D));
}
    
// Filter the input in and obtain output out
int FilterOverlapSave::filter(std::complex<float>* in, std::complex<float>* out)
{
    // Upsampling in by U
    std::complex<float>* up_in = fwdfft->get_in();
    if (U==1) {
        std::copy(in, in+nx, up_in+L-1);
    } else {
        int idx = L-1;
        for (int i=0; i<nx; i++) {
            up_in[idx] = in[i];
            idx += U;
        }
    }

    // Implement overlap-save
    fwdfft->calculate();
    std::complex<float>* transformed = fwdfft->get_out();
    for (int i=0; i<nblocks; i++) {
        int idx = i*fftsize;
#ifdef USE_VOLK
        volk_32fc_x2_multiply_32fc(transformed+idx, transformed+idx, H, fftsize);
#else
        for (int j=0; j<fftsize; j++) {
            transformed[idx++] *= H[j];
        }
#endif
    }
    invfft->calculate();

    std::complex<float>* y = invfft->get_out();
    // Discard overlap output corrupted by circular conv.
    int idx = L-1;
    int oidx = 0;
    std::complex<float>* outbuf = transformed;
    if (D==1) outbuf = out; // put to output array directly
    for (int i=0; i<nblocks-1; i++) {
        std::copy(y+idx, y+idx+M, outbuf+oidx);
        idx += fftsize;
        oidx += M;
    }
    // Last block
    int nlast = Unx-(nblocks-1)*M;
    std::copy(y+idx, y+idx+nlast, outbuf+oidx);
    int nout = this->out_len();
    if (D>1) { 
        // Downsampling filtered by D
        for (int i=0; i<nout; i++) {
            out[i] = outbuf[i*D];
        }
    }
    return nout;
}

// Polyphase filter class
// class constructor
FilterPolyphase::FilterPolyphase(int up, int down, int x_len, int h_len, std::complex<float>* h, int nt) {
    // Set up upsampling and downsampling rates to specified values 
    U = up;
    D = down;

    nx = x_len; // length of input signal
    if (nx%D != 0)
        printf("[WARNING] For proper continuous filtering, input_length must be divisible by D!\n");
    nxD = int(floor(double(nx)/D))+1; // length of down-sampled input signals
    // Set up lengths for the polyphase overlap-save algorithm
    L = h_len; // Length of orginal filter in up-sampled domain
    Lp = int(ceil(double(L)/U/D))+1; // length of polyphase filters
    double LL = (Lp<32) ? 32.0 : double(Lp); // If L too small, use 32
    // Choose FFT size to be 2^m, where m is smallest power that 2^m >= 8*L
    fftsize = int(pow(2.0, ceil(log2(LL)+2)));
    M = fftsize-Lp+1; //Length of data block for each polyphase filter

    // Work out the number of FFT blocks needed per down-sampled branch 
    nblocks = int(ceil(double(nxD)/M));
    // set number of threads to use 
    nthreads = nt;
    
    // Set up forward and inverse FFT objects
    fwdfft = new fft(nthreads, fftsize, nblocks, M, fftsize, D, false);
    invfft = new fft(nthreads, fftsize, nblocks, fftsize, fftsize, U, true);

    // Calculate and save FFTs of polyphase filters' impulse responses
    F = (std::complex<float>*) fftwf_malloc(U*D*fftsize*sizeof(std::complex<float>));
    // Append zeros to the beginning of h to get hh for convenience
    // Also hh[n] = h[n-U*D]
    std::complex<float> hh[fftsize*U*D];
    std::copy(h, h+L, hh+U*D);
    fft fft1time(1, fftsize, 1, 0, 0, false);
    std::complex<float>* f = fft1time.get_in();
    // Note that f[n] = f_{pq}[n-1] below
    for (int p=0; p<U; p++) {
        for (int q=0; q<D; q++) {
            for (int i=0; i<Lp; i++) {
                f[i] = hh[U*D*i+D*p+U*q];
            }
            for (int i=Lp; i<fftsize; i++) {
                f[i] = 0.0;
            }
            fft1time.calculate();
            std::copy(fft1time.get_out(), fft1time.get_out()+fftsize, F+(p*D+q)*fftsize);
        }
    }

    // Set whole fwd->in to 0
    std::fill(fwdfft->get_in(), fwdfft->get_in()+nblocks*D*fftsize, 0.0);
}

// Class destructor
FilterPolyphase::~FilterPolyphase(void) {
    fftwf_free(F);
    delete fwdfft;
    delete invfft;
}

// Set head for next call in continuous filtering
// For continuous filtering length of input must be divisible by D
void FilterPolyphase::set_head(bool reset) {
    std::complex<float>* in = fwdfft->get_in();
    if (reset) {
        for (int q=0; q<D; q++) {
            int startidx = q*nblocks*fftsize;
            int endidx = (q==0) ? startidx+Lp-1 : startidx+Lp;
            std::fill(in+startidx, in+endidx, 0.0);
        }
    } else {
        int destidx = 0;
        int endidx = destidx+nxD+Lp-2;
        int startidx = endidx-Lp+1;
        std::copy(in+startidx, in+endidx, in+destidx);
        for (int q=1; q<D; q++) {
            destidx += nblocks*fftsize;
            endidx = destidx+nxD+Lp-1;
            startidx = endidx-Lp;
            std::copy(in+startidx, in+endidx, in+destidx);
        }
    }
}

// Calculate output length
// Calculate output length
// N.B.: The implementation of the filter method below outputs
// floor(len(input)/D)*U samples. The transient in the beginning is not
// included in the output. This is done so that the filter method 
// may be used to filter a continuous stream of samples.
int FilterPolyphase::out_len(void) {
    return int(floor(double(nx)/D))*U;
}

// Do polyphase filtering
// N.B.: This implementation introduces an additional delay of U samples at output
int FilterPolyphase::filter(std::complex<float>* in, std::complex<float>* out) {
    
    // Step 1 of freq-domain polyphase filtering algorithm
    std::complex<float>* a = fwdfft->get_in();
    int outidx = Lp-1;
    int idx = 0;
    for (int i=0; i<nxD-1; i++) {
        a[outidx++] = in[idx];
        idx += D;
    }
    for (int q=1; q<D; q++) {
        outidx = q*nblocks*fftsize+Lp;
        idx = (D-q)%D;
        for (int i=0; i<nxD; i++) {
            a[outidx++] = in[idx];
            idx += D;
        }
    }
    // Step 2 of freq-domain polyphase filtering algorithm
    fwdfft->calculate();
    // Step 3 of freq-domain polyphase filtering algorithm
    a = fwdfft->get_out();
    std::complex<float>* b = invfft->get_in();
    std::complex<float>* tmp_result = (std::complex<float>*) fftwf_malloc(fftsize*sizeof(std::complex<float>));
    int Fidx = 0;
    for (int p=0; p<U; p++) {
        int startu = p*nblocks*fftsize;
        int uidx = startu;
        int idx = 0;
        for (int i=0; i<nblocks; i++) {
#ifdef USE_VOLK
            volk_32fc_x2_multiply_32fc(b+uidx, a+idx, F+Fidx, fftsize);
#else
            for (int j=0; j<fftsize; j++) {
                b[uidx+j] = F[Fidx+j] * a[idx+j];
            }        
#endif
            uidx += fftsize;
            idx += fftsize;
        }
        Fidx += fftsize;
        for (int q=1; q<D; q++) {
            uidx = startu;
            for (int i=0; i<nblocks; i++) {
#ifdef USE_VOLK
                volk_32fc_x2_multiply_32fc(tmp_result, a+idx, F+Fidx, fftsize);
                volk_32fc_x2_add_32fc(b+uidx, b+uidx, tmp_result, fftsize);
#else
                for (int j=0; j<fftsize; j++) {
                    b[uidx+j] += F[Fidx+j] * a[idx+j];
                }     
#endif
                uidx += fftsize;
                idx += fftsize;
            }
            Fidx += fftsize;
        }
    }
    // Step 4 of freq-domain polyphase filtering algorithm
    invfft->calculate();
    // Steps 5 and 6 of freq-domain polyphase filtering algorithm
    b = invfft->get_out();
    outidx = 0;
    int olen = this->out_len();
    for (int i=0; i<nblocks; i++) {
        for (int j=0; j<M and outidx<olen; j++) {
            int uidx = i*fftsize+j+Lp-1;
            for (int p=0; p<U and outidx<olen; p++) {
                out[outidx++] = b[uidx];
                uidx += fftsize*nblocks;
            }
        }
    }
    // Cleanup
    fftwf_free(tmp_result);
    return olen;
}
```
* <u>FFT size selection heuristic</u>: Choose $m$ to be the minimum
   integer such that $2^m \geq 8 L$ in `FilterOverlapSave` ($2^m \geq
   8 L_p$ in `FilterPolyphase`), and set the FFT size to be
   $2^m$. This selection makes sure that the condition that $2^m \gg
   L$ ($2^m \gg L_p)$. If $L$ ($L_p$) is longer enough, say $L ($L_p$)
   \geq 32$, we also have $L \gg mD$ ($L_p \gg m)$.
* Since the `fft` class is used, it is nice to run the method
`fft:cleanup()` before `main()` exits when using the filter classes.
* <u>Usage example</u>: [`test_filters.cpp`](code:test_filters)

```{admonition} Experiment
:class: hint
1. Study the source code for `FilterOverlapSave` and `FilterPolyphase`.
1. While I have tried to use the FFTW3 interfaces very efficiently and VOLK,
   the other operations required in the frequency-domain filtering
   implementations are not exactly optimized. Try to work on develop
   your own implementation to see if you can beat mine:
   - For example, one may be able to do a better job in structuring the 
     memory storage organization of the input and output, in particular
     in `FilterPolyphase`, to better faciltiate SIMD acceleration.
   - Another possibility for speed-up is to write multi-threaded
     implementations for the "non-FFT" steps in the frequency-domain
     filtering algorithms.
1. Try to see if you can develop a "better" approach to choose
`fftsize`. 
```
