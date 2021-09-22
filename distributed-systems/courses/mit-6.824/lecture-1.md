# Lecture 1 
## Notes
### Introduction
1. What is distributed system?
    1. Set of cooperating computers that are comminicating over a network to get some coherent task done
1. Examples
    1. Large Storage
    1. Big Data Computation
    1. Peer-to-Peer File Sharing 
1. Try everything else before trying distributed systems which is much harder
1. When to build distributed system 
    1. When lot's of parallelism is need (lots of cpu, disk, memory)
    1. Fault Tolerance
    1. Physical Reasons
    1. Security / Isolated 
1. What makes distributed system hard?
    1. Problems that comes with concurrent programming 
    1. Unexpected Failure Patterns (Partial Failures)
    1. Performance
1. Lot of problems in the field is yet to be solved
1. Infrastructure 
    1. Storage
        1. Well defined and useful abstraction
    1. Communication
        1. Tool to build distributed system
    1. Computation 
1. Dream is to develop a interface which acts like a normal file system or computation but it is a very performant and highly fault tolerant distributed system 
1. This dream is rarely fully achieved
1. We need to find the abstractions for the infrastructure 
1. Implementation 
    1.Tools
        1. RPC
        1. Threads
        1. Concurrency
    1. Performance
        1. Scalability
            1. 2x computer = 2x performance (throughput)
            1. Multiple Web Servers & Database example  
    1. Fault Tolerance
        1. Big scale turns problem from very rare events that we don't have worry about that much into a constant problem
        1. Failure has to be really or the response of masking the failure, the ability to proceed without failures just has to built into the design
        1. Build abstraction that hides failures or mask them from the application programmers
        1. Availability
            1. Some number of failures are accepted before it stops responding to the requests
            1. Good available systems are also recoverable 
        1. Recoverability
            1. Non Volatile Storage
                1. Save their state (checkpoint or logs) on disk which can be loaded when the power is back up
                1. Expensive to upgrade
            1. Replication
                1. Two servers with the supposedly identical copy of the system
                1. Management problem is that two servers will accidently drift and be out of sync and will stop being replicas
        1. Consistency 
            1. In the distributed systems there are many replicas, consistency problems appears which is the data should be same on all the replicas 
            1. If client sends the new data to the database and it has two replicas
            1. The first request is successful but at the time of second request the database is crashed 
            1. So there will be outdated value in one replica 
            1. This outdated value can be exposed in the future GET request
            1. Strong Consistency
                1. Expensive communication to implement
            1. Weak Consistency
        1. When we are using replication for fault tolerance we want the replicas to have independent failure probability to have uncorrelated failure
            1. To keep the replicas in the same room and server rack is a bad idea 
            1. If the power to the whole rack goes down service goes down 
            1. Maybe put replicas in different cities or countries for escaping natural disasters in same area
            1. The time takes to talk to a server thousands of miles away is a problem 
            1. Even if it's 20 or 30 ms if we do billions of instruction it adds up to a big number 

### MapReduce 
1. It is system that was originally designed, built and used by Google
1. The paper dates back to 2004
1. Problem they were facing was they were running huge computations on terabytes of data 1. Work included creating index of all of the content of the web or analyzing the link structure of entire web inorder to identify most important pages or the most authorative pages 
1. Building a index is like running a sort on the entire data 
1. They wanted an framework that any programmer can use to do any type of analyis without having the knowledge to code an distributed system   
1. Abstract view of MapReduce 
    1. It assumes that there is some input
    1. The input is then split up into a whole bunch of different files or chunks in some way
    1. Input files can be different websites on the internet
    1. MapReduce then runs the map function on each of the input file
    1. So there is parallelism while applying the function 
    1. Map function takes the input and the output that it produces is list of key value pair 
    1. A simple MapReduce job can be word count MapReduce job 
    1. Goal is to the count the number of occurences of the each word 
    1. Map function output will be word and the value 1
    1. So we run Map function on all the inputs and we get a intermediate output 
    1. Second stage of the computation is the reduce function 
    1. MapReduce will collect all instances from all the map functions each of the key word 
    1. Our example will take the key, value pair and just count them to get the total number of occurences of that word
1. Terminology
    1. The whole computation is called the job 
    1. Single Map or Reduce functions are called the task
    1. Entire job consists of bunch of Map and Reduce task
1. Map and Reduce functions are like ordinary functions written in C++ or any other language that anyone can write
1. Simple Word Count Map Task
```
Map(k, v)
    split v into words
    for each word w
        emit(w, '1')
```
1. Simple Word Count Reduce Task
```
Reduce(k,v)
    emit(len(v))
```
1. We can have multiple MapReduce Job to have chained analysis or for iterative algorithm
    1. Page Rank algorithm is one example of the iterative algorithm
1. MapReduce is great for algorithms where data can be cast in the form of the key, value pairs 
1. What does the emit in map function does?
    1. So there is a master that tells all the worker to perform the map function by giving them the required files 
    1. Then the map function does the computation on the inputs
    1. The emit function then writes the results on the local disk of the workers 
    1. Then the reduce function will query each and every worker in the job to give the respective data that it needs which in turn is approved by the master 
1. What does the emit in reduce function does?
    1. The emit function in the reduce task is different than the emit function in map job
    1. The emit function in the reduce task will save the the results into a cluster file system 
1. Input and the output lives in the cluster file system 
1. We need the flexibility to read any file on any worker so we need some kind of network file system to store input data 
1. Paper talks about the GFS or the Google File System 
1. GFS is a cluster file system and works exactly on the same set of workers that worker that run MapReduce 
1. GFS automatically split up any large file that we store on it across lots of servers in 64 megabyte chunks 
1. Even if we store 1 TB of web crawled data on GFS, it will automatically split it into 64 megabyte chunks and distribute evenly over all the GFS servers
1. When we run MapReduce jobs then the data is already stored in a way that split up evenly across all the servers 
1. Which means that the map workers that is going to launch, if we have a thousand servers it will launch 1000 map workers each reading one of 1000th of the input data 
1. They are going to be able to read the data in parallel from a thousand GFS file serves which will tremendous read throughput 
1. In this paper in 2004, the most constraining bottleneck on the MapReduce system was network throughput 
1. There were thousands of machine talking to each other using ethernet switch 
1. There was a root ethernet switch that all switches talked through which had some amount of total throughput which became the bottleneck
1. Each machine in the paper experiment which was a total of 2000 machines was sharing 50 megabits / second throughput of the root switch 
1. They wanted to avoid things sending on the network
1. One thing they did for this was to run GFS server and MapReduce server on the same set of machine 
1. When the MapReduce job was to be run, the master will cleverly assign the file to be processed to a worker which has the file on its GFS server to reduce transferring files on the network 
1. Because of all the failures, most of the time maps were running on the same machine and stored the data which saved time in moving large amount of data over network
1. Another trick was to store out of map function on the local disk which didn't require immediate transfer of data
1. Transforming the row format of storing to column form of invocations to reduce was called shuffle 
1. Shuffle was the most expensive part of the system where every part of data was to be transferred 
1. Why not use stream processing rather than batch processing 
1. Shuffle is like a sort on the data
1. Input of the sort will be equal to the output of the sort 
1. Output from the reduce tasks were stored on the GFS which again required transferring of data on network 
1. Why not the store the output on the same GFS server on which the reduce will run?
1. They did this but the GFS stores 2 or 3 copies of the files for fault tolerance which resulted in transferring of data on network
1. This became the bottleneck of the system in 2004 
1. In 2020, the modern datacenters are much more faster on network throughput
1. They have multiple root switch where all the switched are connected to multiple root switches
1. Google stopped using MapReduce few years ago but at the ending they were not running the MapReduce jobs on the data of same GFS servers
1. They were happy to transfer data over network which had much throughput than before 


## Topics To Be Researched
1. Link Analyzer used by Google 
1. Page Rank algorithm 