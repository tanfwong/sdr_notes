(code:test_crc_lfsr)=
# `test_crc_lfsr.cpp`

```{code-block}  c++
:lineno-start: 1
:emphasize-lines: 5, 14, 24-32, 42, 45
#include <uhd/utils/safe_main.hpp>
#include <boost/crc.hpp>
#include <cstring>

typedef boost::crc_optimal<32, 0x04C11DB7, 0, 0, false, false> crc_32_lfsr_type; 

int UHD_SAFE_MAIN(int argc, char *argv[]){

    // Instantiate a crc-32 object
    crc_32_lfsr_type crc32;
    unsigned char info[16] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'};
	
    // CRC encode
    crc32.process_bytes(info, 16);

    // Print codeword
    std::cout << "Information:\t"; 
    for (int i=0; i<16; i++) {
        std::cout << std::uppercase << std::hex << (unsigned short) info[i];
    } 
    std::cout << "\nCRC:\t\t" <<  std::uppercase << std::hex << crc32.checksum()  << std::endl;
    // Append info and CRC to form codeword
    unsigned char cw[20];
    memcpy(cw, info, 16);
    unsigned char crc_pt[4];
    boost::uint32_t crc_val = crc32.checksum();
    memcpy(crc_pt, &crc_val, 4);
    // This is needed because of the byte-reversal order of int
    memcpy(cw+16, crc_pt+3, 1);
    memcpy(cw+17, crc_pt+2, 1);
    memcpy(cw+18, crc_pt+1, 1);
    memcpy(cw+19, crc_pt, 1);
    std::cout << "Codeword:\t";
    for (int i=0; i<20; i++) {
        std::cout << std::uppercase << std::hex << (unsigned short) cw[i];
    }
    std::cout << std::endl;
	
    // CRC decode: No error
    std::cout << "\nNo TX error case:\n";
    crc32.reset();
    crc32.process_bytes(cw, 20);

    // Compare calculated checksum to 0
    if (crc32.checksum() == 0) {
        std::cout << "CRC checked => No error\n";
    } else {
        std::cout << "Error detected!\n";
    }

    // CRC decode: Double error
    std::cout << "\nDouble TX error case:\n";
    crc32.reset();
    cw[2] ^= 0x08;
    cw[17] ^= 0x10;
    crc32.process_bytes(cw, 20);

    // Compare calculated checksum to 0
    if (crc32.checksum() == 0) {
        std::cout << "CRC checked => No error\n";
    } else {
        std::cout << "Error detected!\n";
    }

    return EXIT_SUCCESS;
}
```
* In this case, the information and parity-check byte orders are both
  not reflected. We may pass the whole codeword array into the class
  method `process_bytes()`.  This is the same as the hardware linear
  feedback shift register implementation and the second decoding
  method discussed [before](sec:boostcrc). Hence, if the checksum is
  $0$, then there is no error.
```{admonition} Experiment
:class: hint
Build and run the example above to comprare with the implementation in
  [`test_crc.cpp`](code:test_crc).
```
