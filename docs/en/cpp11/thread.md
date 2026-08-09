<div align="right">

[🇺🇸 English](./thread.md) | [🇮🇷 فارسی](../../fa/cpp11/thread.md)

</div>


# thread 
>**A thread is a representation of an execution/computation in a program. In C++11, as in much modern computing, a thread can – and usually does – share an address space with other threads. In this, it differs from a process, which generally does not directly share data with other processes. C++ has had a host of threads implementations for a variety of hardware and operating systems in the past, what’s new is a standard-library threads library.**

>- A new standard of C++11 defined API for threads, and synchronization primitives.
>- As the standard is accepted by all the modern compilers,it is portable to the majority of operating systems.
>- More high-level than pthreads, easier to write clean code.
>- Support for atomicity and memory ordering.
>- **Disadvantages:**
>- Not all synchronization primitives are implemented,e.g. barriers, read-write locks, semaphores...
>- A modern compiler is needed, it is not so well tested as pthreads

## Thread creation - constructor
A thread is launched by constructing a std::thread with a function or a function object:

> `thread thread( Function&& f, Args&&... args );`
> **Parameters:**
>- `f`  function that will be executed by the thread
>- `args` arguments for the start_routine function

```cpp
 #include <thread>
 void f();
 struct F {
         void operator()();
 };
 int main()
 {
         std::thread t1{f};     // f() executes in separate thread
     std::thread t2{F()};   // F()() executes in separate thread
 }
```
## Thread termination
> - It reaches the end of the start_routine
> - It calls return;


**Note:**
- The thread releases its stack during termination.
- Return value
- It is not possible to obtain return code from thread

## Joining threads
![join](./assets/thread.png)

`void thread.join();`
- The function waits for the thread to terminate.
- It is not possible to join one thread more than once.
`bool thread.joinable()` 
- checks if it is possible to join the thread

```cpp
 int main()
 {
         std::thread t1{f};  // f() executes in separate thread
     std::thread t2{F()};    // F()() executes in separate thread
     t1.join();  // wait for t1
     t2.join();  // wait for t2
 }
```

### What happens if the thread is not joined?
- After the thread was terminated, the internal data are stored for further usage.
- The thread.join() function reads this data to provide status information about terminated thread. Afterwards, the function wipes the date out.
- If the thread.join() function is not called we need to let system know that we do not care about the thread and it can release the data.
- It can cause a serious memory leak problem when huge number of threads is used or each thread returns huge structure if those data are not wiped out.

### examples:
In general, we’d also like to get a result back from an executed task. With plain tasks, there is no notion of a return value; std::future is the correct default choice for that. Alternatively, we can pass an argument to a task telling it where to put its result: For example:

```cpp
 void f(vector<double>&);
 struct F {
     vector<double>& v;
     F(vector<double>& vv) :v{vv} { }
     void operator()();
 };
 int main()
 {
     std::thread t1{std::bind(f,some_vec)};  // f(some_vec) executes in separate thread
     std::thread t2{F(some_vec)};        // F(some_vec)() executes in separate thread
     t1.join();
     t2.join();
 }
```
In general, we’d also like to get a result back from an executed task. With plain tasks, there is no notion of a return value; std::future is the correct default choice for that. Alternatively, we can pass an argument to a task telling it where to put its result: For example:


```cpp
void f(vector<double>&, double* res);   // place result in res
struct F {
    vector<double>& v;
    double* res;
    F(vector<double>& vv, double* p) :v{vv}, res{p} { }
    void operator()();  // place result in res
};
int main()
{
    double res1;
    double res2;
    std::thread t1{std::bind(f,some_vec,&res1)};    // f(some_vec,&res1) executes in separate thread
    std::thread t2{F(some_vec,&res2)};      // F(some_vec,&res2)() executes in separate thread
    t1.join();
    t2.join();
    std::cout << res1 << ' ' << res2 << '\n';
}
```

