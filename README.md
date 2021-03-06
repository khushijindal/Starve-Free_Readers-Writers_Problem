# Operating Systems Assignment
# Starve-Free Readers-Writers Problem
#### KHUSHI 18115051
#### Code + Documentation for Starve free Readers-Writers Problem

The Readers-Writers Problem is well-known in computer science. A resource can be accessed by readers, who do not modify the resource, and writers, who can modify the resource. When a writer is modifying the resource, no-one else (reader or writer) can access it at the same time, else another writer could corrupt the resource, and another reader could read a partially modified (inconsistent) value. There are three variations possible for the same: which are defined as first, second and third readers-writers problem respectively. The first gives readers the priority and can result in starvation for writers and the second one gives priority to writers and can lead to starvation for readers. The third Readers-Writers Problem deals with giving a Starve-free environment for both readers and writers. The proposed solution for fair starve-free access for readers and writers both is as discussed below.

## Fundamental Concept:
Give neither priority: all readers and writers will be granted access to the resource in their order of arrival. If a writer arrives while readers are accessing the resource, it will wait until those readers free the resource, and then modify it. New readers arriving in the meantime will have to wait.

## Approach Used: Semaphores
The solution thus proposed, uses Semaphores following a First-In-First-Out strategy to provide access control of the resource to either readers or writers.

### Code for defining Semaphore with FIFO strategy:

```cpp
//SEMAPHORE

// Process Control Block (Node for Queue)
struct Process
{
    Process* next;
    int* id;
    bool state = true; 
    //  state represents whether the process is blocked or active
    //  true represents active while false represents inactive(blocked)

    //  also other things maybe present here, like the return address, subprocesses, etc
}

//Queue which will allow us to form a FIFO semaphore
//Implementing basic queue functions push() and pop()
struct Queue
{
    Process* front, rear;
    
    int* pop()
    {
        // Underflow
        if(front == NULL)
        {
            return -1;            
        }
        
        else{
            int* val = front->value;
            front = front->next;
            if(front == NULL)
            {
                rear = NULL;
            }
            return val;
        }
    }
    
    void* push(int* val)
    {
        Process* blk = new Process();
        blk->id = val;
        blk->state = false;   
          //  the process is blocked before pushing into the blockedQueue
          //  in reality this is done using system calls
            
        if(rear == NULL)
        {
            front = rear = n;
            
        }
        else
        {
            rear->next = blk;
            rear = blk;
        }
    }
    
}

// Semaphore following FIFO strategy
// structure of the semaphore, as you can see we have linked it to a queue
// this queue would store the list of proessess waiting to acquire the semaphore
struct Semaphore
{
  int value = 1;
  Queue* Q = new Queue();
}
 
//wait()
void wait(Semaphore *S,int* process_id)
{
  S->value--;
  if(S->value < 0)
  {
     S->Q->push(id); 
     //This function will block the proccess until it's woken up.
     //The process will remain in the waiting queue 
     //till it is waken up by the wakeup() system calls
     //This is a type of non-busy waiting
  }
}

//Analogous of wakeup() system call
void wakeup(Process* p) {
    p->state = true;
}
//  In reality, a system call is used instead of this function
//  I have used this function as an analogy of the syscall

//signal()  
void signal(Semaphore *S)
{
  S->value++;
  if(S->value <= 0){
  int* next_process = S->Q->pop();
  wakeup(next_process); 
  //this function will wakeup the process with the given pid using system calls
  }
}

```
A FIFO semaphore is thus implemented. A queue is used to manage the waiting processes. The process gets blocked after pushing itself onto the queue and is woken up in FIFO order when some other process releases the semaphore.

### Initial Values of the Semaphores:
The solution uses three semaphores. The <b>turn</b> semaphore is used to specify whose chance is to next enter the critical section. The process with this semaphore gets the next chance to enter the critical section. The <b>rwt</b> semaphore is required to access the critical section. <b>rmutex</b> semaphore is required to change the readcount variable which maintain the number of active readers.
```cpp
// INITIALIZATION

int read_count = 0;                     
//Variable representing the number of readers 
//currently executing the critical section

Semaphore turn = new Semaphore();        
//Semaphore representing the order in which the writers and 
//readers are requesting access to critical section

Semaphore write_mutex = new Semaphore();         
//Semaphore required to access the critical section
//by the writer for mutual exclusion

Semaphore read_mutex = new Semaphore();     
//Semaphore required to change the read_count variable
//this variable prevents the conflicts while changing the read_count variable
//conflicts might have arose when two readers access the variable simulataneously
```

### Reader Process Code:
Following is the code for the reader process. I have made a function for modularising the implementation. This function would be called in a when any new process asking for reading access to the critical section arrives.
```cpp
//Reader Code 
// this function would be called everytime a new reader arrives

do{
//<ENTRY SECTION>
       wait(turn,process_id);              
       //waiting for its turn to get executed
       
       wait(read_mutex,process_id);           
       //requesting access to change read_count
       
       read_count++;                       
       //update the number of readers trying to access critical section 
       
       // if it is the first reader then request access to critical section
       //requesting  access to the critical section for readers
       if(read_count==1)                   
         wait(write_mutex);
         
       signal(turn);                      
       //releasing turn so that the next reader or writer can take the token
       //and can be serviced
       
       signal(read_mutex);                    
       //release access to the read_count
       
//<CRITICAL SECTION>
       
//<EXIT SECTION>
       wait(read_mutex,process_id)            
       //requesting access to change read_count 
       
       read_count--;                       
       //a reader has finished executing critical section so read_count decrease by 1
       
       //if all the reader have finished executing their critical section
       //releasing access to critical section for next reader or writer
       if(read_count==0)                   
        signal(write_mutex);
        
       signal(read_mutex);                    
       //release access to the read_count  
       
//<REMAINDER SECTION>
       
}while(1);
```
### Writers Process Code:
Following is the code for the writer process. The writer code that would be called every time a new writer arrives.
```cpp
do{
//<ENTRY SECTION>
      wait(turn,process_id);              
      //waiting for its turn to get executed
      
      wait(write_mutex,process_id);               
      //requesting  access to the critical section
         
//<CRITICAL SECTION>
//Write the data in this "critical section"

//<EXIT SECTION>
      signal(turn,process_id);            
      //releasing turn so that the next reader or writer can take the token
      //and can be serviced
                                       
      signal(write_mutex)                        
      //releasing access to critical section for next reader or writer

//<REMAINDER SECTION>

}while(1);
```
## Explanation of its working:
The starve-free solution works on this method : Any number of readers can simultaneously read the data without any issue. The "write_mutex" semaphore ensures that only a single writer can access the critical section at any moment. Once a writer has come, no new process that comes after it can start reading, ensuring mutual exclusion. Also, when the first reader tries to access the critical section it has to acquire the "read_mutex" lock to access the critical section thus ensuring mutual exclusion between the readers and writers. 

Before accessing the critical section any reader or writer have to first acquire the "turn" semaphore which uses a FIFO queue for the blocked processes. Thus as the queue uses a FIFO policy, every process has to wait for a finite amount of time before it can access the critical section thus meeting the requirement of bounded waiting. This ensures that any new process that comes after this (be it reader or writer) will be queued up on "turn".

Suppose processes come as RRRWRWRRR. Now, by our method the first three readers will first read. Then, when the next process which is a writer takes the turn semaphore, it won't give it up until it's done. Also, it acquires the "write_mutex" semaphore. Now, suppose by the time this writer is done writing, the next writer has already arrived. However, it will be queued up on the turn semaphore. Thus, it won't get to start writing before the process before it, the reader process has completed. Thus, the reader process will get done and then the writer process will again block any more processes from starting and so on.

The code is structured so that there are no chances for deadlock and also the readers and writers takes a finite amount of time to pass through the critical section and also at the end of each reader writer code they release the semaphore for other processes to enter into critical section.

Thus, it is pretty clear how we manage to make a starve-free solution for the readers-writers problem with a FIFO semaphore. We can say that all processes will be handled in a FIFO manner. Readers will allow other readers in the queue to start reading with them but writers will block all other processes waiting in the queue from executing before it finishes. In this way, we can implement a reader-writer solution where no process will have to indefinitely wait leading to starvation.

## References:
- https://rfc1149.net/blog/2011/01/07/the-third-readers-writers-problem/
- https://en.wikipedia.org/wiki/Readers%E2%80%93writers_problem
- Abraham Silberschatz, Peter B. Galvin, Greg Gagne - Operating System Concepts
- https://arxiv.org/abs/1309.4507
