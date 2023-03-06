# Starve Free Reader-Writer Problem

The Reader-Writer Problem is a synchronisation problem in which two types of processes (reader and writer) are trying to access the critical section (the code containing data which is shared among different process).
The processes of type reader read the data from the critical section. So multiple reader processes can access critical section at the same time.
The proccesses of type writer update/change the data in the critical section. So, no two writers can access the critical section at the same time.
Even reader process and writer process cannot access the critical section at the same time.

## Solution Intuition 

Semaphores are used to synchronise the processes such that no two writer processes or one writer and reader processes are accessing critical section at the same time. And if a situation arise when the above case happens then the process will go into the waiting queue.
The normal solution uses two semaphores. When a reader enters the critical section it acquires the semaphore which is needed for writer process to enter it's critical section. And a writer needs a semaphore to enter a critcal section. Therfore, niether a reader and writer can enter in the critical section simultaneously nor two writer can enter critical section section simultaneously.
The above solution solves the synchronisation problem but creates starvation problem, which happens when one reader process enters critical section and after that a writer process comes then it will go into the wating states, now until or unless all the reader processes are executed the writer process will stay in the wating queue which leads to starvation.
Therefore, instead of two, three semaphores are used such that if a writer comes after reader, then it will be given priority as FIFO(First In First Out). Hence, eliminating the chance of starvation of writer.

## Solution Implementation

### Process Control Block

```cpp
struct process{
    int Process_id; // Id of the process
    bool state_is_active;// if true then the process is active else blocked;
    process* next_process = NULL; //stores the next process
    /*
    the process control block contains other information also but i have mentioned only those which are required for this solution
    */
}
```
### Blocking Queue

```cpp
struct blocking_queue{
    process *Q_front, *Q_back; // stores the first and last process of the queue
    int size = 0 // size of the queue
    void push_to_queue(process &P){ // the process is taken as the input
        P->state_is_active = false; // the process entering the queue is blocked
        size++;
        if(size==1){
            Q_front = P;
            Q_back = P;
        }
        else {
            Q_back->next_process = P;// the process is added to the end of the queue
            Q_back = P;
        }
    }
    proecess* pop_from_queue(){
        process* front_process = Q_front;// the first process of the queue is removed
        front = front->next;
        size--;
        return front_process;
    }
}
```
The bocking queue is First In First Out
### Semaphore 
```cpp
struct semaphore{
    int sem = 1;
    semaphore (int n ){
        sem = n;
    }
    blocking_queue blockqueue = new blocking_queue;
    void wait(process* p){
        sem--;
        if(sem<0){
            blockqueue.push_to_queue(p);// process is added to the blocking queue
        }
    }
    void wake_up(process* p ){
        p->state_is_active= true;
    }
    void signal(){
        sem++;
        if(sem<=0){
            process* waking_process = blockqueue.pop_from_queue();
            wake_up(waking_process);
        }
    }
}
```

### Global Declarations 
```cpp
semaphare* writer_sem = new semaphore(1);// the semaphores are initialized with one which is the number of readers that can be in critical section at a time
semaphore* reader_sem = new semaphore(1);
semaphore* in_sem  = new semaphore(1);// without this semaphore the write will suffer from starvation
int reader_count = 0;// this counts the number of readers in the critical section

```
### Reader Code
```cpp
while(true){
// process_p is a process which is currently active
    in_sem.wait(process_P);// this ensures if writer is in the queue for wating then writer is given priority according to FIFO so that writer is not starving
    reader_sem.wait(process_P);
    reader_count++;
    if(reader_count==1){
        // if reader count is 1 that means the first reader just started execution so it aquires the writer semaphore
       writer_sem.wait(process_P); 
    }
    reader_sem.sigal();
    in_sem.signal();

    /* 

        Critical Section
        
    */

    reader_sem.wait(process_P);
    reader_count--;
    if(reader_count==0){
        writer_sem.signal(); // when read count becomes zero then writer can access the critical section therefore writer semaphore is released
    }
    reader_sem.signal();
}
```
### Writer Code
```cpp
while(true){
    in_sem.wait(process_P);
    writer_sem.wait(process_P);
    in_sem.signal();
    /* 
    
        Critical Section 
        
    */
    writer_sem.signal();
}
```

## Correctness of the Solution
### Mutual Exclusion
For Mutual Exclusion either one writer can be accessing the critical section or one or more readers can be accessing the critical section. This is ensured by `reader_sem` semaphore which calls wait and acquires the `writer_sem` semaphore until there are atleast one reader in critical section. Therefore, writer cannot enter the critical section. Similary, if writer is in critical section then `writer_sem` semaphore will be required for reader to access the critical section which will ensure that is writer is in critical section then reader cannot access it's critical section. The `in_sem` semaphore also ensures that writer and reader are not in critical section simultaneously.
### Progress
There is no cyclic access of resources therefore, the system cannot enter the deadlock state. Once a process completes the critical section then it signals the semaphores such that other process can enter the critical section.
### Bounded Waiting
The generic solution with two semaphores causes starvation is readers come one after the other leaving writer to starve. But in this solution the third semaphore `in_sem` is ensuring FIFO. So, once Writer process arrives it is added to the queue, and the readers coming after the writer are also added to the queue such that until or unless the writer completes it's execution of the critical section, the other reader process after that reader cannot enter the critical section.

##### Hence, the solution satisfies all the requiremnets. So this can be said a solution to **Starve-Free Reader-Writer Problem**.
