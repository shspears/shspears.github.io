---
layout: "post"
title: "concurrency"
date: "2020-05-24 09:57"
---

# C++ Concurrency notes

> Most of the notes here were taken from Udemy's [Complete Modern C++](https://www.udemy.com/course/beg-modern-cpp/) course, authored by Umar Lone. Great course!

---


- Thread [Creation](#thread-creation)
- [Sharing Resources](#sharing-resources-between-threads) between Threads
- Thread [Functions](#thread-functions)
- [Task Based](#task-based-concurrency) Concurrency
- [std::async](#std--async)
- [std::future](#std--future) and some [functions](#std--future-functions)
- [Launch Policies](#launch-policies)
- [std::promise](#std--promise)
- [Exceptions](#exceptions)


---
### Thread Creation
- std::thread
    - Since C++11
    - Accepts a callable (call back, functor, lambda expression) as a constructor argument
    - The callable is exectuted in a separate Thread.
    - The Constructor does not wait for the thread to start; it returns immediately

- To ensure the parent process does not end before the child, use threadName.join() at the end of the parent scope.
- .detach() will detach the chaild from the parent. Once detached, the child thread can no longer join (rejoin) the parent
- .joinable() is a way to see if you can use .join() at the end of the parent thread, to wait for the child to finish before finishing itself.



- when passing arguments to the callable within the thread constructor - you can pass by reference (if the callable accepts by reference) using the std::ref() helper function
- std::cref() is used for const arguments to be passed to the callable by (const) reference.



> An example of passing arguments to a thread

```cpp
    #include <iostream>
    #include <thread>
    #include <list>
    #include <string>


    std::list<int> g_Data;
    const int SIZE = 5000000;


    //Creating a custom String class to demonstrate passing arguments by ref
    class String
    {
    public:
        String()
        {
            std::cout << "String()" << std::endl;
        }
        String(const String&)
        {
            std::cout << "String(const String&)" << std::endl;
        }
        String& operator=(const String&)
        {
            std::cout << "operator=(const String&)" << std::endl;
            return *this;
        }
        ~String()
        {
            std::cout << "~String()" << std::endl;
        }
    };

    void Download(String &file)
    {
        for (int i = 0; i < SIZE; i++)
        {
            g_Data.push_back(i);
        }

        std::cout << "Download Complete" << std::endl;
    }

    int main()
    {
        String fileName;
        std::cout << "main; User started an operation" << std::endl;

        //we can pass arguments (even by reference) if the function has arguments (and args that are pass by ref)
        //to pass arguments by ref, we use std::ref(arg)
        //to pass arguments by const ref, we use std::cref(arg)
        std::thread thDownloader(Download, std::ref(fileName));
        //thDownloader.detach();

        std::cout << "main: User started another operation " << std::endl;

        //Always check to make sure the thread is joinable before trying to join it!
        if (thDownloader.joinable())
        {
            thDownloader.join(); // wait for child to finish
        }

        system("Pause");

        return 0;
    }

```

---
### Sharing Resources Between Threads

- .lock and std::lock_guard
- To use .lock(), you lock just before the critical area and .unlock() just after the critical section
- This can be an issue, in the event the critical section throws an exception before it's able to unlock()
- lock_guard() uses the RAII idiom by creating the lock in the constructor and unlocking in its destructor - which gets called as soon as the lock_guard goes out of scope.
    - Performance trade offs of using lock_guard are not known (by me) at this time.


- Global data (yuck) or object attributes are a way to share data between threads
-



```cpp

    #include <iostream>
    #include <thread>
    #include <list>
    #include <string>
    #include <mutex>


    std::list<int> g_Data;
    std::mutex g_Mutex;

    const int SIZE = 10000;

    void Download(std::string &file)
    {
        for (int i = 0; i < SIZE; i++)
        {
            //g_Mutex.lock(); //Not safe
            std::lock_guard<std::mutex> mLock(g_Mutex); //safer!
            //Now we don't need to lock / unlock
            g_Data.push_back(i);
            //Lets hope there isn't an exception here!@ deadlock!
            //g_Mutex.unlock();
        }

        std::cout << "Download Complete" << std::endl;
    }

    void Download2(std::string& file)
    {
        for (int i = 0; i < SIZE; i++)
        {
            std::lock_guard<std::mutex> mLock(g_Mutex); //safer!
            //std::lock_guard creates the lock on its constructor
            //and unlocks in its destructor. RAII!!
            //g_Mutex.lock(); //not safe
            g_Data.push_back(i);
            //g_Mutex.unlock();
        }

        std::cout << "Download2 Complete" << std::endl;
    }

    int main()
    {
        std::string fileName{ "awesome.mp3" };

        std::cout << "main; User started an operation" << std::endl;

        //we can pass arguments (even by reference) if the function has arguments (and args that are pass by ref)
        //to pass arguments by ref, we must use std::ref(arg)
        //to pass arguments by const ref, we must use std::cref(arg)

        std::thread thDownloader(Download, std::ref(fileName));
        std::thread thDownloader2(Download2, std::ref(fileName));

        //thDownloader.detach();

        std::cout << "main: User started another operation " << std::endl;

        //Always check to make sure the thread is joinable before trying to join it!
        if (thDownloader.joinable())
        {
            thDownloader.join(); // wait for child to finish
        }
        if (thDownloader2.joinable())
        {
            thDownloader2.join(); // wait for child to finish
        }
        std::cout << "List Size: " << g_Data.size() << std::endl;

        //system("Pause");

        return 0;
    }

```

---
### Thread functions

- Nested namespace within std - std::this_thread
    - std::this_thread::get_id() - get the pid of the thread this is called from
    - std::this_thread::sleep_for(std::chrono::seconds(1))
- std::thread::hardware_concurrency() - get the number of cores of the CPU the program is running on
- thread.native_handle() - returns a HANDLE on Windows, pThread on Linux
- SetThreadDescription() - in Windows.h - used to name a thread (for debugging)

- chrono_literals :
    - using namespace std::chrono_literals;
    - now you can use std::this_thread::sleep_for(1s); //same as std::chrono::seconds(1)


```cpp

HANDLE handle = thread1.native_handle();
SetThreadDescription(handle, L"MyThread");

```

```cpp

    #include <iostream>
    #include <thread>
    #include <Windows.h>

    void Process()
    {
        auto x = std::this_thread::get_id();
        std::cout << " Thread ID of Process(): " << x << std::endl;
    }

    int main()
    {
        auto x = std::this_thread::get_id();
        std::cout << " Thread ID of main(): " << x << std::endl;
        std::thread t1(Process);

        std::cout << "Thread ID: " << t1.get_id() << std::endl;

        auto handle = t1.native_handle();
        SetThreadDescription(handle, L"MyThread"); //label thread for easier debugging

        //Linux used pthread_setname

        int cores = std::thread::hardware_concurrency();
        //auto th = std::thread::
        std::cout << "Number of cores on this machine: " << cores << std::endl;

        t1.join();

        return 0;
    }



```



---
### Task Based Concurrency

- A task is a function that is executed in its own thread

- using the async function in the future header

```cpp

    #include <future>
    ...
    //std::async(callable, argsToBePassed,..); //This executes synchronously..!
    std::future<void> result = std::async(callable, argsToBePassed,..);
    //std::future<int> -> the type of future object is the return type of the callable!!

```

##### std::async

- part of high-level concurrency
- Executes a callable object of a function in a separate thread
- returns a std::future object that provides access to the result of the callable
- include the header file < future>

- Two overloaded constructors:
    - future< return_type> async(Callable, args)
    - future< return_type> async(launch policy, Callable, args)

- Launch Policy
    - std::launch::deferred - Task executes synchronously
    - std::launch::async - Task executes asynchronously
    - ***Always include your launch policy explicitly*** as specific compilers may not execute a new thread otherwise
    - If new thread cannot be created (with the launch policy), std::system_error is thrown

- Arguments are passed (by default) to the callable by value
    - To pass by reference use ***std::ref()***
    - To pass by const reference use ***std::cref()***

- now you can get the return value from the task (callable function) as a std::future< return type of the callable > object!!

---

##### std::future

- used for communications between threads
- has a shared state that can be accessed in a different thread
- created through a std::promise object
    - created by std::async, that directly returns a future object
    - std::promise is the "input channel"
    - std::future is the "output channel"
- The thread that reads the shared state will wait until the future is ready (with the shared state).

##### std::future functions

- .wait() is similar to .get() except it does not return a value; it simply waits for the async to complete.
- .wait_for() returns an enum.
    - std::future_status::deferred - async is deferred, and process is blocking
    - std::future_status::ready - ready to use .get() to get the return value of the callable
    - std::future_status::timeout - timed out... task is still running
- .wait_until() - accepts a time point.

```cpp

  std::future< std::string> result = std::async(std::launch::async, CallableFunc, std::ref(CallableArg));

  using namespace chrono_literals;
  auto timePoint = std::chrono::system_clock::now();
  timePoint += 1s; //add a second to the current time
  //auto status = result.wait_for(4s);
  auto status = wait_until(timePoint);

  switch(status)
  {
      case std::future_status::deferred :
            break;
      ... //ready, and timeout

  }
```
---
##### Launch Policies

- std::launch has two policies:
    - std::launch::async - which automatically executes the callable in a new thread
    - std::launch::deferred - which waits to execute the callable until the future object calls its get() function.
- If the future object calls the get() function and the callable has not completed, the program will wait until the return type is available
- You can (if running a large while loop) use the function ._Is_ready() (of the future object) until the return value is ready to use.



```cpp

    std::future<std::string> opOutput = std::async(std::launch::async, Operation);

     using namespace std::chrono_literals;
     std::this_thread::sleep_for(1s);
     std::cout << "Main thread continues..." << std::endl;

     std::string result{ "Not Done.. :(" };
     bool notDone = true;

     while (notDone)
     {
         std::cout << "In while loop... " << std::endl;
         if (opOutput._Is_ready())
         {
             //std::cout << "Output seems to be valid... but, we're stuck at output.get() until async() returns..." << std::endl;
             result = opOutput.get();
             std::cout << "Async thread returned the string: " << result << std::endl;
             notDone = false;
         }
         else
         {
             std::cout << result << std::endl;
         }

     }

```

---

##### std::promise

- Provides a way to store a value or an exception
- shared state
- This state can be accessed later from another thread through a future object
- promise / future are two endpoints of a shared communication channel
- one operation stores the value in a ***promise*** and the other operation will retrieve it through a ***future*** asynchronously.
- These operations are synchronized, and therefore thread-safe.
- ***a promise object can be used only once***
    - to use it more than once you have to reinitialize the promise object

```cpp

std::promise<std::string> stringData;
...
stringData = std::promise<std::string>();
```

- std::future and std::promise provide a good way to pass data to and from threads.
- The following shows an example of passing data from an async callable to a detached thread

```cpp

#include <iostream>
#include <future>
#include <thread>
#include <string>

using namespace std::chrono_literals;
bool keepRunning = true;


void navString(std::promise<std::string>& data)
{
  //This function will create a promise (string)
    std::string navData{ "This is a nav string" };
    std::cout << "Entering NavString" << std::endl;
    std::this_thread::sleep_for(500ms);
    std::cout << "Leaving NavString" << std::endl;
    data.set_value(navData);

}

void run(std::promise<std::string> &navData)
{
  //This function receives the promise data and displays it
    while (keepRunning)
    {

        auto data = navData.get_future();
        for (int i = 0; i < 10; i++)
        {
            std::cout << ".";
            std::this_thread::sleep_for(500ms);
        }
        std::cout << "done the run loop" << std::endl;
        auto print = data.get();
        std::cout << "Nav data: " << print << std::endl;
    }
}

int main()
{
    std::cout << "Main thread starts." << std::endl;

    std::promise<std::string> navData; //promise data
    std::thread mainThread(run, std::ref(navData));
    mainThread.detach();
    int count = 0;

    while (true)
    {
        //asynchronously start the navString callable to create the promise
        std::future<void> opOutput = std::async(std::launch::async, navString, std::ref(navData));

        std::this_thread::sleep_for(1s);

        navData = std::promise<std::string>(); //reinitialize the promise to reuse again.

        //the following is just a way to test how to break out after a certain amount of calls.
        if (count++ == 10)
        {
            keepRunning = false;
            break;
        }
    }

}

```


##### exceptions

- A promise can propagate an exception between threads.
- Using a try catch block around setting the (promise) data. You have to set an ***exception pointer***

```cpp
    try
    {
        if(x != valid)
          throw std::runtime_error{"Data not valid"};
        else
          data.set_value(x);
        ...
    }catch(std::exception &ex)
    {
        data.set_exception(std::make_exception_ptr(ex));
    }

```

- If an exception is thrown it will be available in the future object
- by using the .get() function of the future object, if the exception was thrown in the promise (as above), it will be thrown once .get() is called.
- Make sure to also put a try catch block around this block as well.
