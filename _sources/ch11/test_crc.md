(code:test_crc)=
# `test_crc.cpp`

```{code-block}  c++
:lineno-start: 1
:emphasize-lines: 9, 14, 23-27, 36-46
#include <uhd/utils/safe_main.hpp>
#include <boost/crc.hpp>
#include <cstring>
 
 
int UHD_SAFE_MAIN(int argc, char *argv[]){
    
    // Instantiate a crc-32 object
    boost::crc_32_type crc32;
    // Info byte array
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
    memcpy(cw+16, &crc_val, 4);
    std::cout << "Codeword:\t";
    for (int i=0; i<20; i++) {
        std::cout << std::uppercase << std::hex << (unsigned short) cw[i];
    }
    std::cout << std::endl;

    // CRC decode: No error
    std::cout << "\nNo TX error case:\n";
    crc32.reset(); // Reset for new CRC calculation
    crc32.process_bytes(cw, 16); // Calculate CRC checksum

    // Compare calculated checksum with RX checksum
    boost::uint32_t rx_crc;
    memcpy(&rx_crc, cw+16, 4);
    if (rx_crc == crc32.checksum()) {
        std::cout << "CRC checked => No error\n";
    } else {
        std::cout << "Error detected!\n";
    }
    std::cout << "RX CRC:\t\t" << std::uppercase << std::hex << rx_crc << std::endl;
    std::cout << "Calculated CRC:\t" << std::uppercase << std::hex << crc32.checksum() << std::endl;

    // CRC decode: Double error
    std::cout << "\nDouble TX error case:\n";
    crc32.reset();
    // Add double error
    cw[2] ^= 0x08;
    cw[17] ^= 0x10;
    crc32.process_bytes(cw, 16);

    // Compare calculated checksum with RX checksum
    memcpy(&rx_crc, cw+16, 4);
    if (rx_crc == crc32.checksum()) {
        std::cout << "CRC checked => No error\n";
    } else {
        std::cout << "Error detected!\n";
    }
    std::cout << "RX CRC:\t\t" << std::uppercase << std::hex << rx_crc << std::endl;
    std::cout << "Calculated CRC:\t" << std::uppercase << std::hex << crc32.
    checksum() << std::endl; 

    return EXIT_SUCCESS;
}
```
* In this decoding process here, we enter only the portion o fthe RX
codeward byte array corresponding to the information part in the
codeword into the class method `process_bytes()`. This corresponds to
the first decoding method discussed [before](sec:boostcrc). We have to
compare the calculated checksum to the RX one to see whether there is
any error.
```{admonition} Experiment
:class: hint
Build and run the example above to see how Boost.CRC does CRC encoding
and decoding. 
```
