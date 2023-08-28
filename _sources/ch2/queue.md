(code:queue)=
# `queue.hpp`

## Source code
```{code-block}  c++
:lineno-start: 1
// University of Florida EEL6528
// Tan F. Wong
// Jan 17, 2021

#pragma once

#include <mutex>

// Define template for thread-safe FIFO 
template<class T>
class MutexFIFO {
  public:
    void push(T);
    bool pop(T&);
    int size();
    void lock();
    void unlock();
    std::list<T> queue; // May make this private if want max protection
  private:
    std::recursive_mutex mtx;
};

template <class T>
void MutexFIFO<T>::push(T entry) {
    std::lock_guard<std::recursive_mutex> scoped_lock(mtx);
    queue.push_back(entry);
}

template <class T>
int MutexFIFO<T>::size(void) {
    std::lock_guard<std::recursive_mutex> scoped_lock(mtx);
    return queue.size();
}

template <class T>
bool MutexFIFO<T>::pop(T& entry) {
    std::lock_guard<std::recursive_mutex> scoped_lock(mtx);
    bool is_not_empty = not queue.empty();
    if (is_not_empty) {
        entry = queue.front();
        queue.pop_front();
    }
    return is_not_empty;
}

template <class T>
void MutexFIFO<T>::lock(void) {
    mtx.lock();
}

template <class T>
void MutexFIFO<T>::unlock(void) {
    mtx.unlock();
}
```
