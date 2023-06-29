(code:age_mutex_caller)=
# `age_mutex_caller.cpp`

## Source code
```{code-block}  c++
:lineno-start: 1
:emphasize-lines: 22-36, 54-57
// University of Florida EEL6528
// Tan F. Wong
// Jan 14, 2021

#include <uhd/utils/safe_main.hpp>
#include <uhd/utils/thread.hpp>
#include <boost/program_options.hpp>
#include <mutex>
#include <thread>
#include <chrono>
#include <csignal>
#include <sstream>
 
namespace po = boost::program_options;

// Interrupt signal handlers
static bool stop_signal_called = false;
void sig_int_handler(int){
    stop_signal_called = true;
}

// Caller-enable locking
class SharedPrinter {
  private:
    std::mutex mtx;
  public:
    void print(std::string message) {
        std::cout << message;
    }
    void lock() {
        mtx.lock();
    }
    void unlock() {
        mtx.unlock();
    }
}; 

// worker function to print age
void print_age(std::string name, int age, bool nice, SharedPrinter* printer) {

    std::ostringstream message;
    if (nice) {
        // nice printing
        if (age < 0) {
            message << "From Thread #" << std::this_thread::get_id() << ": " << name <<  "'s age is unknown." << std::endl;
        } else {
            message << "From Thread #" << std::this_thread::get_id() << ": " << name << " is " << age << " years old." << std::endl;
        }
    } else {
        // not nice printing
        message << "Thread #" <<  std::this_thread::get_id() << ": " << std::endl << "Name = " << name << std::endl << "Age = " << age << std::endl;
    }
    while (not stop_signal_called) {
        printer->lock();
        printer->print(message.str());
        printer->print(message.str());
        printer->unlock();
        std::this_thread::sleep_for(std::chrono::seconds(1)); // Just don't go too fast
    }
}

int UHD_SAFE_MAIN(int argc, char *argv[]) {
    // Set main thread priority to highest
    uhd::set_thread_priority_safe(1.0, true);
 
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

    // Instantiate the SharedPrinter object
    SharedPrinter printer;
    
    // Spawn 10 worker threads
    int num_threads = 10;
    std::thread worker[num_threads];
    for (int i=0; i<num_threads; i++) {
        worker[i] = std::thread(&print_age, name, age, vm.count("nice-print"), &printer);
    }
        
    // Keep running until CTRL-C issued
    while (not stop_signal_called) {
        std::this_thread::sleep_for(std::chrono::milliseconds(1)); // sleep for 1ms to make sure can catch CTRL-C
    }

    // Join all worker threads
    for (int i=0; i<num_threads; i++) {
        if (worker[i].joinable()) worker[i].join();
    }

    std::cout << std::endl;
    return EXIT_SUCCESS;
}
```
* Lines 54-57 show a typical use of this caller-ensured locking method
  to block other threads from access the object during the two
  consecutive calls to the `SharedPrinter::print()` method.

## Build
* Need to link the `uhd`, `boost_program_options`, `pthread` libraries when compiling:
  ```
  g++ age_mutex_caller.cpp -o age_mutex_caller -luhd -lboost_program_options -lpthread
  ```
## Usage Example
```
$ ./age_mutex_caller --nice-print --name=Tom -a 25
```
```{admonition} Experiment
:class: hint
Run the example above to see what happens! Compare the results with
   those obtained running [age_mutex.cpp](code:age_mutex).
```

