# Carrier Synchronization

* Carrier synchronization is the process that matches the frequency of
the locally generated carrier reference (just called local oscillator or
LO here for easy reference) at the RX to that of the RX carrier
signal. The latter may be different from the nominal carrier frequency
due to component mismatch, drift, and physical channel phenomena
(e.g., Doppler shift).
* In addition, if coherent demodulation is to be performed, the phase
of LO must also match that of the RX carrier signal.
* Often, known training symbols, referred to as **pilot symbols**,
are periodically inserted into the data symbol sequence in order to
aid achieving carrier synchronization. One may use these pilot symbols
to estimate the exact carrier frequency and phase at the RX. Or one
may use a class of closed-loop circuitry called **phase-locked loops
(PLLs)** to track the frequency and phase of the carrier. Usually,
demodulated data symbols are fed back to drive the PLLs in addition to
the pilot symbols resulting in **decision-directed PLLs**.
* When no pilot symbols are available, the carrier needs to be tracked
using **non-decision-directed PLLs**.
