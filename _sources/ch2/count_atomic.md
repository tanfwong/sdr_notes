(code:count_atomic)=
# `count_atomic.cpp`

## Source code
```{code-block}  c++
:lineno-start: 1
:emphasize-lines: 36, 38, 41, 81, 86
// University of Florida EEL6528
// Tan F. Wong
// Jan 14, 2021

#include <uhd/utils/safe_main.hpp>
#include <uhd/utils/thread.hpp>
#include <boost/program_options.hpp>
#include <atomic>
#include <mutex>
#include <thread>
#include <chrono>
#include <csignal>
#include <sstream>
 
namespace po = boost::program_options;

// Interrupt signal handlers
static bool stop_signal_called = false;
void sig_int_handler(int) {
    stop_signal_called = true;
}

// Internal locking
class SharedPrinter {
  private:
    std::mutex mtx;
  public:
    void print(std::string message) {
        mtx.lock();
        std::cout << message;
        mtx.unlock();
    }
}; 

// worker function that waits for its turn and then increments count
void wait_and_count(int id, int n, std::atomic<unsigned long>& count, SharedPrinter& printer) {
    while (not stop_signal_called) {
        if ((count.load()/10)%n == id) {
            // It is this thread's turn 
            std::ostringstream message;
            message << "Thread #" << id << " increments count to " << ++count << std::endl;
            printer.print(message.str());
            std::this_thread::sleep_for(std::chrono::seconds(1)); // Just don't go too fast
        } else {
            // Not this thread's turn
            std::this_thread::yield();
        }
    }
}

int UHD_SAFE_MAIN(int argc, char *argv[]) {
    // Set main thread priority to highest
    uhd::set_thread_priority_safe(1.0, true);
 
    //variables to be set by po
    int num_threads;
 
    //setup the program options
    po::options_description desc("Allowed options");
    desc.add_options()
        ("help", "help message")
        ("num-threads,n", po::value<int>(&num_threads)->default_value(10), "Number of threads")
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
    
    // Initialize atomic varaible storing the count
    std::atomic<unsigned long> count(0);

    // Spawn num_threads worker threads
    std::thread worker[num_threads];
    for (int i=0; i<num_threads; i++) {
        worker[i] = std::thread(&wait_and_count, i, num_threads, std::ref(count), std::ref(printer));
    }
        
    // Keep running until CTRL-C issued
    while (not stop_signal_called) {
        std::this_thread::yield(); // This is more CPU efficient
    }

    // Join all worker threads
    for (int i=0; i<num_threads; i++) {
        if (worker[i].joinable()) worker[i].join();
    }

    std::cout << std::endl;
    return EXIT_SUCCESS;
}
```

```{admonition} Experiment
:class: hint
1. Study the source code . Compile and run it. Can you explain what the code does?
1. Why do we still need to use the mutexed `SharedPrinter`object here?
1. Note the use of `std::this_thread::yield()` and the way of passing
   references in the function inside the `std::thread` constructor.
```
