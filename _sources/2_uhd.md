# Some Programming Prelims

## UHD
* UHD is the C++ device driver for USRP radios.
* It provides APIs to set up, query, and control a USRP radio and to transfer signal samples between the USRP radio and a host computer.
* The version of UHD that we will use in class (installed on `wing-radio-a` and `wing-radio-b`) is [UHD 3.15.0.0](https://files.ettus.com/manual_archive/v3.15.0.0/html/index.html). Note that it is NOT the lastest stable version of UHD, which can be found [here](https://files.ettus.com/manual/).


### UHD_SAFE_MAIN macro and Termination trick
* [UHD_SAFE_MAIN](./uhd_safe_main.md) is a macro for `main()` that catches and prints out an exception, in case it is thrown, to `stderr`. It is not necessary, but good, to use it. 
* In most cases, we want our `main()` and other functions to keep running until we decide otherwise. Thus we need a way to terminate the program. The simplest way to do so is to issue a CTRL-C interrupt. See below for a sample code to catch the CTRL-C interrupt and terminate `main()`.
* <u>Example</u>: 

```c++
#include <uhd/utils/safe_main.hpp>
#include <csignal>

// Interrupt signal handler
static bool stop_signal_called = false;
void sig_int_handler(int)
{
    stop_signal_called = true;
}

int UHD_SAFE_MAIN(int argc, char *argv[])
{
    ...
	// Set the signal handler
    std::signal(SIGINT, &sig_int_handler);
    ...
    // Keep running until CTRL-C issued
    while(not stop_signal_called){
        ...
    }
    return EXIT_SUCCESS;
}
```


### Boost `program_options` Library
* [Boost](http://www.boost.org/) is a huge collection of C++ libraries that help us do many common application tasks. We will mainly use the Boost libraries for option parsing, printing, and sometimes threading. The version of boost installed installed on `wing-radio-a` and `wing-radio-b` is [Boost 1.71.0](https://www.boost.org/doc/libs/1_71_0/).
* In particular, Boost's [`program_options` library](https://www.boost.org/doc/libs/1_71_0/doc/html/program_options.html) will come in handy when parsing program options. 
* <u>Example</u>: [age.cpp](age.md)

