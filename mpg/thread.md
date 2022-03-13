# thread

## Thread States:
+ **Waiting**: Thread is stopped and waiting for something in order to continue. Reasons like: ***waiting for the hardware (disk, network), the OS (system calls) or synchronization calls (atomic, mutexes)***. These types of latencies are a root cause for bad performance.
+ **Runnable**: Thread wants time on a core so it can execute its assigned machine instructions. If you have lots of threads that want time, then threads have to ***wait longer to get time***. Also, ***the individual amount of time any given thread gets is shortened***, as more threads compete for time. This kind of scheduling latency can also be a cause of bad performance.
+ **Executing**: This means thread has beeen placed on a core and is executing its machine instructions. The work related to the application is getting done. This is what everyone wants.

## Types Of Work
+ **CPU-Bound**: Never put thread in waiting states. Constantly make calculations.
+ **IO-Bound**: Causes threads to enter into waiting states. This is work that consists in requesting access to a resource over the network or making system calls into the OS. 

## Context Switching
The physical act of swapping threads on a core is called a context switch. Context switches are considered to be expensive because it takes times to swap threads on and off a core. Once a thread moves into a waiting state, another thread in a runnable state is there to take its place. This allows the core to always be doing work. Don't allow a core to go idle if there is work (threads in a runnable state) to be done.
