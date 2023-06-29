# Digest of FCC Regulations Part 15

## General Regulations 

* [FCC Regulations Part 15](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15)
 "sets out the regulations under which an intentional, unintentional,
or incidental radiator may be operated without an individual license."
We will perform experiments using the USRP radios within the purview of
Part 15.

* The following sections in Part 15 are most relevant to us: 
  - [Subpart A](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-A?toc=1): General 
    - [Section 15.5](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-A/section-15.5): General conditions of operation
    - [Section 15.23](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-A/section-15.23): Home-built devices 
  - [Subpart C](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C?toc=1): Intentional Radiators
    - [Section 15.205](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C/section-15.205): Restricted bands of operation
    - [Section 15.209](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C/section-15.209): Radiated emission limits; general requirements
    - [Section 15.245](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C/subject-group-ECFRf45d172319b1f81/section-15.245): Operation within the bands 902-928 MHz, 2435-2465 MHz, 5785-5815 MHz, 10500-10550 MHz, and 24075-24175 MHz.
    - [Section 15.247](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C/subject-group-ECFRf45d172319b1f81/section-15.247): Operation within the bands 902–928 MHz, 2400–2483.5 MHz, and 5725–5850 MHz.
    - [Section 15.249](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C/subject-group-ECFRf45d172319b1f81/section-15.249): Operation within the bands 902–928 MHz, 2400–2483.5 MHz, 5725–5875 MHZ, and 24.0–24.25 GHz.
  - [Subpart E](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-E?toc=1): Unlicensed National Information Infrastructure Devices
* Other sections that we won't use but you may want to take a look at:
  - [Subpart F](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-F?toc=1): Ultra-Wideband Operation
  - [Subpart G](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-G?toc=1): Access Broadband Over Power Line (Access BPL)
  - [Subpart H](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-H?toc=1): (TV) White Space Devices

## Industrial, Scientific, and Medical bands
* We will conduct our radio experiments in the 902-928 MHz, 2400-2483.5 MHz, and 5725-5850 MHz Industrial, Scientific, and Medical (ISM) bands.
* Note that the SBX daughterboards can operate in the 902-928 MHz and 2400-2483.5 MHz bands, while the CBX daughterboards can operate in the 2400-2483.5 MHz and 5725-5850 MHz bands.
* [Section 15.245](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C/subject-group-ECFRf45d172319b1f81/section-15.245),  [Section 15.247](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C/subject-group-ECFRf45d172319b1f81/section-15.247), and [Section 15.249](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C/subject-group-ECFRf45d172319b1f81/section-15.249) allow low-power, non-licensed transmission in these bands. Section 15.247 concerns us the most because it mainly specifies the restrictions for communication signals.
* Below is a summary of transmission allowed by  [Section 15.247](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C/subject-group-ECFRf45d172319b1f81/section-15.247) (some of the wordings below are direct quotes from the FCC sections above):

### Frequency hopping 
* Any two hopping channel carrier frequencies must be separated by at least the maximum between 25 kHz and the 20 dB bandwidth of the hopping channel.
* The system must hop to pseudo-random channel frequencies at the system hopping rate. Each frequency must be used equally on the average by each transmitter.
* It is not required to employ all available hopping channels during each transmission, provided all other requirements are satisfied.
* The incorporation of intelligence within a frequency hopping system that permits the system to recognize other users within the spectrum band so that it individually and independently chooses and adapts its hopsets to avoid hopping on occupied channels is permitted. The coordination of frequency hopping systems in any other manner for the express purpose of avoiding the simultaneous occupancy of individual hopping frequencies by multiple transmitters is not permitted.

* **902-928 MHz band:**
  - If the 20 dB bandwidth of the hopping channel is less than 250 kHz, the system shall use at least 50 hopping frequencies and the average time of occupancy on any frequency shall not be greater than 0.4 seconds within a 20 second period. The maximum peak conducted output power shall not exceed 1 W.
  - If the 20 dB bandwidth of the hopping channel is 250 kHz or greater, the system shall use at least 25 hopping frequencies and the average time of occupancy on any frequency shall not be greater than 0.4 seconds within a 10 second period. The maximum peak conducted output power shall not exceed 0.25 W.
  - The maximum allowed 20 dB bandwidth of the hopping channel is 500 kHz.
  - The maximum peak conducted (for use with TX antennas of less than 6 dBi gain) output power may be 0.25 W if 25-49 hopping channels are used, or 1 W if $\geq$ 50 hopping channels are used.

* **2400-2483.5 MHz band:**
  - There should be at least 15 channels. The average time of occupancy on any channel shall not be greater than 0.4 seconds within a period of 0.4 seconds multiplied by the number of hopping channels employed.
  - The system may avoid or suppress transmissions on a particular hopping frequency provided that a minimum of 15 channels are used. 
  - If at least 75 non-overlapping hopping channels are used, the maximum peak conducted output power may be 1 W. Otherwise it may be 125mW.
  - If the system's output power is no greater than 125 mW,  any two hopping channel carrier frequencies may be separated by the maximum between 25 kHz and $\frac{2}{3}$  of the 20 dB bandwidth of the hopping channel.

* **5725-5850 MHz band:**
  - At least 75 hopping frequencies must be used. 
  - The maximum 20 dB bandwidth of the hopping channel is 1 MHz. 
  - The average time of occupancy on any frequency shall not be greater than 0.4 seconds within a 30 second period.
  - The maximum peak conducted output power may be 1 W.

### Digital modulation
* The minimum 6 dB bandwidth must be at least 500 kHz.
* The maximum peak conducted output power or the maximum conducted output power shall not exceed 1 W. Maximum Conducted Output Power is defined as the total transmit power delivered to all antennas and antenna elements averaged across all symbols in the signaling alphabet when the transmitter is operating at its maximum power control level. The average must not include any time intervals during which the transmitter is off or is transmitting at a reduced power level. If multiple modes of operation are possible (e.g., alternative modulation methods), the maximum conducted output power is the highest total transmit power occurring in any mode.
* The power spectral density conducted to the antenna shall not be greater than 8 dBm in any 3 kHz band during any time interval of continuous transmission.
* Devices using digital modulation techniques in the 5725-5850 MHz bands are specified under [Subpart E](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-F?toc=1).

### Antenna gain
* The conducted output power limit above is based on the use of antennas with directional gains that do not exceed 6 dBi. 
* If transmitting antennas of directional gain greater than 6 dBi are used, the conducted output power shall be reduced below the stated values by the amount in dB that the directional gain of the antenna exceeds 6 dBi. 
* Other further restrictions may also apply. See [Section 15.247(c)](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C/subject-group-ECFRf45d172319b1f81/section-15.247) for details.

### Out-of-band radiation
* In any 100 kHz bandwidth outside the frequency band in which the system is operating, the radio frequency power that is produced by the intentional radiator must be at least 20 dB below that in the 100 kHz bandwidth within the band that contains the highest level of the desired power, provided the transmitter demonstrates compliance with the peak conducted power limits. 
* If the transmitter complies with the conducted power limits based on the use of RMS averaging over a time interval, the attenuation required shall be 30 dB.
* Attenuation below the general limits specified in Section 15.209 is not required. 
* Radiated emissions which fall in the restricted bands, as defined in Section 15.205, must comply with the radiated emission limits specified in [Section 15.209](https://ecfr.federalregister.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-C/section-15.209).

