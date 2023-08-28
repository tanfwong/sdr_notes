(code:age_fifo)=
# `age_fifo.cpp`

## Source code
```{code-block}  c++
:lineno-start: 1
:emphasize-lines: 75, 88, 39, 93
// University of Florida EEL6528
// Tan F. Wong
// Jan 17, 2021

#include <uhd/utils/safe_main.hpp>
#include <uhd/utils/thread.hpp>
#include <boost/program_options.hpp>
#include <thread>
#include <csignal>
#include <sstream>
#include "queue.hpp"

namespace po = boost::program_options;

// Interrupt signal handlers
static bool stop_signal_called = false;
void sig_int_handler(int){
    stop_signal_called = true;
}

// worker function to generate age message and push it to FIFO
void print_age(std::string name, int age, bool nice, MutexFIFO<std::string>& fifo) {

    // Set thread priority to highest
    uhd::set_thread_priority_safe(1.0, true);
    std::ostringstream message;
    if (nice) {
        // nice printing
        if (age < 0) {
            message << "Main thread popped message from Thread #" << std::this_thread::get_id() << ": " << name <<  "'s age is unknown." << std::endl;
        } else {
            message << "Main thread popped message from Thread #" << std::this_thread::get_id() << ": " << name << " is " << age << " years old." << std::endl;
        }
    } else {
        // not nice printing
        message << "Main thread popped message from Thread #" <<  std::this_thread::get_id() << ": " << std::endl << "Name = " << name << std::endl << "Age = " << age << std::endl;
    }
    while (not stop_signal_called) {
        fifo.push(message.str());
        std::this_thread::sleep_for(std::chrono::seconds(1)); // Just don't go too fast
    }
}

int UHD_SAFE_MAIN(int argc, char *argv[]){

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
    if (vm.count("help")){
        std::cout << desc << std::endl;
        return EXIT_SUCCESS;
    }

    std::signal(SIGINT, &sig_int_handler);

    // Construct FIFO to hold all messages
    MutexFIFO<std::string> fifo;

    // Spawn 10 worker threads
    int num_threads = 10;
    std::thread worker[num_threads];
    for (int i=0; i<num_threads; i++) {
        worker[i] = std::thread(&print_age, name, age, vm.count("nice-print"), std::ref(fifo));
    }
    
	std::string message;
	 
    // Keep running until CTRL-C issued
    while (not stop_signal_called) {
        if (fifo.pop(message)) 
            std::cout << message << " (" << fifo.size() << " messages left in FIFO)" << std::endl;
        else
            std::cout << " FIFO is empty!" << std::endl;
        // The sleep_for time here determines the service rate 
        std::this_thread::sleep_for(std::chrono::milliseconds(80)); 
    }

    // Join all  worker threads
    for (int i=0; i<10; i++) {
        if (worker[i].joinable()) worker[i].join();
    }

    std::cout << std::endl;
    return EXIT_SUCCESS;
}
```
* Ten worker threads push messages to the FIFO queue. Each thread
  pushes one message into the FIFO queue each second. Hence, the total
  arrival rate is 10 messages per second. 
* The main thread pops messages out of the FIFO queue. The service rate is
  determined by the `sleep_for` time on line 93.

```{admonition} Experiment
:class: hint
1. Study the source code . Compile and run it. What results do you observe?
1. Why don't we need to use the mutexed `SharedPrinter` object here?
1. Vary the `sleep_for` time on line 93 to see its effects on the queue size.
```
