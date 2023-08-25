(sec:multithread)=
# Elementary Multi-threading

* To implement a communication system in software, we often need to
  run functions that send and/or receive signal samples to and/or from
  the USRP as well as many other processing functions asynchronously
  and simultaneously.
* This can be done by multi-threaded programming, in which different
  functions are implemented in different threads. Exactly how many
  treads to use and which functions should be implemented in which
  threads are typical design questions.
* Multi-threading imposes overheads. If you are pushing the CPU to its
  limits and concern about overheads, a common rule of thumb is to
  have the maximum number of simultaneous CPU-hogging threads to be
  roughly the same as the number of processors (or cores)
  available. In our Linux boxes, all the processors contain 4 cores
  with hyper-threading support. Hence the maximum of number of
  simultaneous CPU-hogging threads should be about 8.
* Generally we should have a main thread, the one starts `main()`, and
  other spawned threads. We will use the
  [`std::thread`](http://www.cplusplus.com/reference/thread/thread/)
  class to create, launch, and manage threads. 
  ```{caution} 
  `std::thread` is available in C++11 or after. 
  ```
  ```{admonition} Dig deeper
  You may also use the somewhat more powerful
  [Boost.Thread](https://www.boost.org/doc/libs/1_74_0/doc/html/thread.html)
  library to do multi-threading. 
  ```

## Create, launch, and join threads
* The simplest way to create and launch a thread is to instantiate a
  `std::thread` class object by passing to its constructor a function
  pointer pointing to the function that the thread is to
  execute. Other input arguments of the function can also be passed to
  the constructor. See the code example below.
* After a thread is completed, we should **join** the thread back to
  the main thread for clean termination. This is done by the using
  `std::thread::join()` method, which will block the calling thread
  until the thread represented by the thread object has completed. To
  make sure that a thread can be joined, we may check if the thread is
  "joinable" using the `std::thread::joinable()` method. A thread is
  not "joinable" if it has been already joined or detached.
* One may use the namespace `std::this_thread` to invoke a set of
  functions that access the current thread. For example, calling the
  `std::this_thread::sleep_for()` method will cause the current thread
  to pause for a specified amount of time. We should always try to do
  that whenever possible in a thread to vacate CPU resources to other
  running threads.
* <u>Example</u>: [`age_threads.cpp`](code:age_threads) 
   
## UHD thread priority
* The priority of the main thread can be set using the
  `uhd::set_thread_priority_safe()` method in `main()`. The same
  method can be used to set the thread priority in the function that a
  spawned thread executes. The `uhd::set_thread_priority_safe()`
  method is a UHD wrapper to set thread priority using
  [Pthreads](https://computing.llnl.gov/tutorials/pthreads/) in
  UNIX-based systems. The `gcc` compiler, in most of such systems,
  implements the `std::thread` class based on Pthreads. Hence, you may
  also use the same method to set priority of threads that you create
  using `std::thread`. This also applies if you use the Boost.Thread
  library instead. 
* The two input parameters of `uhd::set_thread_priority_safe()` are:
  - `priority`: a `float` in [0,1] with 0 and 1 respectively meaning lowest and highest priority for the thread (default = 0.5)
  - `realtime`: a `bool` specifying whether real-time scheduling should be enabled or not (default = `true`)
* Setting `priority` to 1 and `realtime` to `true` typically gives the thread
  highest priority. This is what we want to do for threads that
  implement time-critical processing, such as sending samples from the
  host to the USRP and grabbing samples back to the host from the
  USRP. 
  ```{note}
  Real-time scheduling means a certain way to
  schedule a fair and constant access to the CPU. Such a restriction
  may not necessarily reduce the run time of a program.
  ```
* <u>Example</u>: [age_threads.cpp](code:age_threads) 

## Thread synchronization with mutex
* If you run [age_threads.cpp](code:age_threads), the console printout
  is not what you have expected. Despite the messages printed out by
  different threads sometimes come out cleanly, they often garble
  together. This is because the threads are running asynchronously
  without regard of the existence of each other. The characters of the
  message strings from different threads may be put into the buffer of
  `std::cout` by the respective `<<` operators in the threads in some
  unpredictable order, garbling the output.
* The situation can be worse when multiple threads are writing to the
same memory location that holds a pointer to an object. A race
condition may occur and the content of memory location may become
unpredictable. Any subsequent reference to the object may cause the
program to crash! The technical terminology to describe this situation is that
such an implementation is not **thread-safe**.
  ```{note} 
  Strictly speaking, both `std::cout` and the `<<` operator are
  thread-safe because the implementation indeed prevents multiple
  threads from writing to the buffer of `std::cout`
  simultaneously. In fact,  [age_threads.cpp](code:age_threads)
  doesn't crash. However, this thread-safe property applies only on a
  character-by-character basis; hence we still have the messages from
  different threads garbled together.
  ```
  ```{caution}
  **Neither `uhd::tx_streamer::send()` nor
  `uhd::rx_streamer::recv()` is thread-safe.**
  ```
* In order to make [age_threads.cpp](code:age_threads) work in the way
  as we want it to, we need to provide a form of synchronization among
  all the threads that a thread can get exclusive access to
  `std::cout`, blocking other threads from using `std::cout` before
  the thread is done streaming its whole message to `std::cout`.
* **Mutexes** provide a basic mechanism for achieving this form of
  synchronization among threads. We will use the `std::mutex` class to
  construct mutexes. 
  ```{admonition} Dig deeper 
  You may also use the more versatile `boost::mutex` class from the
  [Boost.Thread](https://www.boost.org/doc/libs/1_71_0/doc/html/thread.html)
  library. 
  ```
* There are many ways to use mutexes for thread synchronization.
   The following three simple ways are pretty much all needed for most
   of our purposes.
  ```{admonition} Dig deeper 
  See this [tutorial](http://www.boost.org/doc/libs/1_71_0/doc/html/thread/synchronization.html)
  for a short introduction and {cite}`williams2019` for a more
  detailed treatment. 
  ```
### Internal locking
* Consider 
  ```c++
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
  ```
  which will be used in place of `std::cout` in
  [age_threads.cpp](code:age_threads)
* Suppose that a thread calls `SharedPrinter:print()`. The
  `std::mutex::lock()` method in `SharedPrinter:print()` gives
  ownership of the mutex ` SharedPrinter::mtx` to the thread, and
  blocks other threads from accessing the object. After the whole
  message is streamed to `std::cout`, the method
  `std::mutex::unlock()` releases the current thread's ownership of
  the mutex, allowing other threads to access the `SharedPrinter`
  object again. As a result, only one thread can stream a message to
  `std::cout` at a time, and no other thread can stream to `std::cout`
  until the current thread finishes streaming its whole message.
* Note that `std::mutex::lock()` is a blocking call. It will return
  until the mutex is unlocked by its current owner and the calling
  thread obtains ownership of the mutex. The method
  `std::mutex::try_lock()` attempts to obtain the mutex's ownership
  for the current thread without blocking and returns `true` if
  succeeds.
* One may also use a **scoped lock** in the definition of `SharedPrinter`
in place of the explicit calling of `std::mutex::lock()` and
`std::mutex::lunlock()`:
  ```c++ 
  class SharedPrinter {
    private:
      std::mutex mtx;
    public: 
      void print(std::string message) {
          std::lock_guard<std::mutex> scoped_lock(mtx); 
          std::cout << message; 
      }
  };
  ```
  The object `scoped_lock`'s constructor and destructor locks and
  releases the mutex, respectively.
* Example code showing how to use `SharedPrinter`:
  [age_mutex.cpp](code:age_mutex)

### Caller-ensured locking
* Consider the following implementation of the `SharedPrinter` class:
  ```c++
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
  ```
  where the mutex locking and unlocking methods are exposed to the caller
  function.  The class method `SharedPrinter::print()` is **not** thread-safe
  and the caller is responsible for blocking other threads from
  accessing it.
* Example code showing how to use this caller-ensured locking version
  of `SharedPrinter`: [age_mutex_caller.cpp](code:age_mutex_caller)

### Internal + Caller-ensured locking
* We may obtain the best of both methods above by the following
  implementation of the `SharedPrinter` class:
  ```c++
  class SharedPrinter {
    private:
      std::recursive_mutex mtx;
    public:
      void print(std::string message) {
          std::lock_guard<std::recursive_mutex> scoped_lock(mtx);
          std::cout << message;
      }
      void lock() {
          mtx.lock();
      }
      void unlock() {
          mtx.unlock();
      }
  };
  ```
  where the class method `SharedPrinter ::print()` is now thread-safe.
* Note that we have to use `std::recursive_mutex` in this implementation
  to allow for the possibility of the same mutex being locked
  repeatedly internally and by the caller. 
* ```{admonition} Experiment
   :class: hint
   1.  Use this new implementation of `SharedPrinter` in
       [age_mutex_caller.cpp](code:age_mutex_caller). Compile and run
       your modified code to observe the results.
   1. Instead of `std::recursive_mutex` in the implementation of
       `SharedPrinter` above, use `std::mutex` to construct the scoped
       clock. Compile and test the resulting code. Explain what happens.
       ```{hint}
       If the program hangs, don't panic. You may press CTRL-Z to stop
       it. Then run `ps` to find out its process number and `kill -9
       <process number>` to remove it.
       ``` 
  ```

## Passing information/signal between threads using atomic objects
* Information and data may be passed between threads via shared
  objects. The methods to access these shared objects should be
  thread-safe. That can be achieved by employing the mutex-based
  techniques discussed before.
* When we want to share a variable of fundamental type, such as
  `bool`, `int`, `float`, and *etc*, we may effectively achieve
  locking by declaring the variable as `atomic` using the
  `std::atomic<>` template. An atomic object is free from data races.
  If one thread writes to an atomic object while another thread reads
  from it, the behavior is well-defined.
* The basic methods to write to and read from an atomic variable are
  `std::atomic::store()` and `std::atomic::load()`, respectively. One
  may also use the `=` operator in place of
  `std::atomic::store()`. Other operators, such as `++` and `--`, and
  operation methods are also available for atomic objects. See
  [this link](https://www.cplusplus.com/reference/atomic/atomic/) for
  details. 
  ```{admonition} Dig deeper 
  An atomic object may be used to synchronize access to other
  non-atomic objects near it in different threads by specifying
  different memory orders. See {cite}`williams2019` for details. By
  default, `std::atomic::store()` and `std::atomic::load()` employ the
  strictest memory order. The `=` operator is equivalent to
  `std::atomic::store()` with the strictest memory order.
  ```
* One may also use atomic objects to pass signals from one thread to
  others in order to trigger their operations. 
* <u>Example:</u> [`count_atomic.cpp`](code:count_atomic)
  ```{admonition} Dig deeper 
  A more sophisticated, CPU-efficient way to pass signals between
  threads is to use mutexes together with condition variables. See
  this
  [simple example ](https://www.cplusplus.com/reference/condition_variable/condition_variable/)
  and {cite}`williams2019` for a more detailed exploration.  
  ```
  
## Thread-safe FIFO queue
* We will often encounter the common scenario in which the output
  generated by a thread needs to be passed on to another thread for
  further processing. Since all threads are running asynchronously, we
  need to provide thread-safe buffering between the data-generating
  and data-receiving threads.
* A simple example of such thread-safe buffering is the FIFO queue
  implemented in the `MutexFIFO` class template defined in the header
  file [`queue.hpp`](code:queue).
* <u>Example code</u> that shows how to use this FIFO
  queue: [age_fifo.cpp](code:age_fifo)
* One should make sure that the arrival rate to the FIFO queue should
  be smaller than its service rate. Otherwise the size of the queue
  will grow and eventually cause memory problems.
