(code:age)=
# `age.cpp`

## Source code
```{code-block}  c++
:lineno-start: 1
:emphasize-lines: 9, 23-34
// University of Florida EEL6528
// Tan F. Wong
// Jan 8, 2021

#include <uhd/utils/safe_main.hpp>
#include <boost/program_options.hpp>
#include <csignal>
 
namespace po = boost::program_options;

// Interrupt signal handlers
static bool stop_signal_called = false;
void sig_int_handler(int) {
    stop_signal_called = true;
}
 
int UHD_SAFE_MAIN(int argc, char *argv[]) {
 
    //variables to be set by po
    std::string name;
    int age;
 
    //setup the program options
    po::options_description desc("Allowed options");
    desc.add_options()
        ("help", "help message")
        ("name,n", po::value<std::string>(&name)->default_value("This person"), "The person's name")
        ("age,a", po::value<int>(&age)->default_value(-1), "The person's age")
        ("nice-print", "Print out the info nicely")
    ;
 
    po::variables_map vm;
    po::store(po::parse_command_line(argc, argv, desc), vm);
    po::notify(vm);
 
    //print the help message
    if (vm.count("help")) {
        std::cout << desc << std::endl;
        return EXIT_SUCCESS;
    }

    std::signal(SIGINT, &sig_int_handler);

    // Keep running until CTRL-C issued
    while (not stop_signal_called) {
        if (vm.count("nice-print")) { 
            // nice printing
            if (age < 0) {
                std::cout << name <<  "'s age is unknown." << std::endl;
            } else {
                std::cout << name << " is " << age << " years old." << std::endl;
            }
        } else {
            // not nice printing
            std::cout << "Name = " << name << std::endl;
            std::cout << "Age = " << age << std::endl;
        }
        sleep(1); // Just don't print too fast
    }
 
    std::cout << std::endl;
    return EXIT_SUCCESS;
}
```


## Build
* Need to link the `uhd` and `boost_program_options` libraries when compiling:
  ```
   $ g++ age.cpp -o age -luhd -lboost_program_options
  ```


## Usage examples
```
$ ./age --help
Allowed options:
  --help                           help message
  -n [ --name ] arg (=This person) The person's name
  -a [ --age ] arg (=-1)           The person's age
  --nice-print                     Print out the info nicely
```

```
$ ./age
Name = This person
Age = -1
Name = This person
Age = -1
^C
```

```
$ ./age --nice-print
This person's age is unknown.
This person's age is unknown.
This person's age is unknown.
^C
```

```
$ ./age --nice-print --name=Tom -a 25
Tom is 25 years old.
Tom is 25 years old.
Tom is 25 years old.
Tom is 25 years old.
^C
```
