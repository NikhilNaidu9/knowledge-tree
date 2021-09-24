# Lecture 2
## Notes
1. Why use Go?
    1. Good supports for threads, locking and synchronization between threads 
    1. It has a convinient RPC (remote procedure call package)  
    1. It is a type-safe and memory-safe
    1. It is garbage collected 
    1. Combination of threads and garbage collected is very important 
    1. Things that go wrong in a non-garbage collected 
        1. In C++ if we use threads, it's always a puzzle and requires a bunch of bookkeeping to figure out when the last thread that's using a shared object has finished using that object because only then we can free the object 
        1. We end up writing quite a bit of code, it's like manually the programmer it's bunch of code to manually do reference counting or something in order to figure out when the last threads stopped using an object 
        1. This problem completely goes away if we use garbage collection
    1. It's much simpler than C++
        1. For e.g. if there is an type or an error in the program, the error message that outputed by the compiler is so complicated
        1. It's usually not worth to figure out what error message meant 
        1. It's much quicker to look at the line number and try to guess what error message would be 
### Threads 
1. Reason why we care so much about threads 
    1. It's the main tool used to manage concurrency in programs 
    1. Concurrency is a particular interest in distributed programming because it's often the case that one program actually needs to talk to a bunch of other computers 
    1. Client may talk to many servers, servers may be serving requests at the same time on behalf of many different clients
    1. We need all this without lot of complex programming, threads are the answer for this type of situation
1. In Go, threads are called as Go routines
1. Way we can think of threads is that there is one program and one address space 
1. In a serial program in that address space there is only one thread of execution executing code 
    1. One program counter
    1. One set of registers
    1. One stack
    1. This all are describing the current state of the execution
1. In a threaded program, we could have multiple threads 
1. Multiple threads represetnts separate program counter, set of registers and stacks
1. Threads can refer to each others stacks if they knew the right addresses, we typically don't do that
1. In Go, even the main program when we start up the program and it runs the main that's also just a go routine and can do everything that a go routine can do 
1. Different parts of the program to be in it's own point in the activity which we refer as I/O concurrency 
1. This is because of the historical reason, in the old days one thread was waiting to read from the disk and the second thread is writing on the disk or send a message on the network and wait for reply
1. For your use case, concurrency will mean a program has launched many RPC to many servers and waiting for reply 
1. We assign one thread to launch RPC to a server and wait until the reply comes and start processing after 
1. I/O concurrency is overlapping of the progress of different activities and allowing one activity is waiting and other activities can proceed
1. Another big reason to use threads is multi-core parellelism 
1. If we have a compute heavy job that needs a lot of CPU cycles, wouldn't it be nice to if you could have one program that could use CPU cycle on all of the cores of the machine
1. If we write a multi-threaded Go program and if you launch multiple Go routines, doing something compute intensive like sit there in a loop and you know compute digits of pi or something, then up to the limit of the number of cores in the physical machine, threads will run truly in parallel 
1. If we launch two threads instead on one we will be able to use twice as many CPU cycles per second
1. Third reason is that we need to do something in the background periodically
1. E.g. There will be a master server which will check periodically whether their worker are still alive because one of them died, we want to launch another machine 
1. When we send a message every second to the worker to check whether it's alive can be implemented in the go routine that just sits in the loop that sleeps for a second and does the periodic thing and then sleeps for a second again 
1. Is the overhead worth it?
1. What if we had no threads, then what would have we done to implement something like server talking to many clients?
    1. It turns out that there is another line or kind of strucuturing this kind of programming which we can call it as asynchronous programming or event-driven programming 
    1. General structure of the event-driven programming is usually that it has a single thread and single loop and what that loop does is sits there and wait for any input or event might trigger processing so an event might be arrival of the request from the client or timer going off 
    1. Lot of Windows-system are written like event-driven programming like waiting for clicks 
    1. It's a little bit of pain to program this 
    1. In this way we get I/O concurrency but we don't get CPU parallelism 
1. What is the difference between processes and threads?
    1. Process is single program that we are running and sort of single address space, single bunch of memory
    1. Inside of process, we might have multiple threads 
    1. Running a go program creates one process, when go creates routines, they sit inside the process
    1. OS only cares about the process, it has nothing to do with what happens inside the process
    1. When we have multiple process there is a isolation between them i.e different memory
    1. Within one program the threads can share memory and can synchronize with channels and use mutexes and stuff but between processes there is just no interaction 
1. When a context switch happens, does it happens for all threads?
1. Locks
1. Coordination 
    1. Channels 
    1. sync.Cond
    1. Wait Group
1. Deadlock
1. Closures


## Topics To Be Researched
1. Multi-core Parellelism 
1. Context Switching 
1. RACE Condition