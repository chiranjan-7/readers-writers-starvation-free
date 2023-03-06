# Readers-Writers-Starvation-Free
## Problme Description 
There are two different user types in the Readers-Writers Issue, which is a synchronisation issue : 

- **READERS** - They are able to read data from shared files or shared memory locations. Several readers can simultaneously access these shared files or memory locations. 

- **WRITERS** - They have the ability to edit shared files and data stored in memory. 
When a writer is currently working on a file and a READER or another WRITER attempts to access it, synchronisation issues may arise.

## Simple Solution (with Starvation):
By using semaphores and bearing in mind their ideal operating circumstances, we may resolve the fundamental problem of synchronisation. The semaphores used are **`mutex`** and **`wrt`**. And we keep a global variable **`readcount`** to keep account of the number of READERS reading the shared data
- **Starvation Problem** - If new readers are entering while the writer is waiting for the readers to release the wrt lock, the writer will continue to be in the waiting stage. Starvation results as a result.
## Starvation free solution :
Implementing a queue priority system for the readers and writers is the answer. The readers come first before the writers. This indicates that the writer must wait until all readers have finished using the resource before granting access to a reader. In a similar fashion, all readers must wait until the writer has finished seeking access to the resource. 

The readers and writers must be lined up in a FIFO (First In First Out) arrangement to ensure impartiality. This implies that earlier readers and writers will be given preference over later ones. 

**Pseudo Code:** 
```
//*******************SEMAPHORE FIFO PSEUDOCODE************//
struct Semaphore {
  int value = 1;
  Queue *Q = new Queue();
  void sem_init(int data){
    value=data;
  }
  void wait(int pid){
    value--;
    if(value<0){
      Q->push(pid);
      block();
    }
  }
  void post(){
    value++;
    if(value<=0){
      int pid=Q.front();
      Q.pop();
      wakeup(pid);
    }
  }
}
//*********************************************************//

//************* Initialization of Global Variables ****************//

allow: semaphore(1)
allownext: semaphore(1)
writer_wait: semaphore(0)
allowreader: int(0)
popreader: int(0)
waitwriter: bool(false)

//************* READER function pseudocode:****************//

void *reader(void* id){
  //ENTRY SECTION
  allow.wait()
  allowreader=allowreader+1
  allow.signal()

  //CRITICAL SECTION
  Read DATA

  //EXIT SECTION
  allownext.wait()
  popreader=popreader+1
  if(waitwriter==true && allowreader==popreader)
  then writer_wait.signal()
  allownext.signal()
}

//*************WRITER function pseudocode:****************//

void *writer(void* id){
  //ENTRY SECTION
  allow.wait()
  allownext.wait()
  if(allowreader==popreader)
  then
    allownext.signal()
  else{
    waitwriter=true
    allownext.signal()
    writer_wait.wait()  
    waitwriter=false
  }

  //CRITICAL SECTION
  Process and Modify Data

  //EXIT SECTION
  allow.signal()
}

```
**Semaphores Used:** 
- `finish` : Prevents 2 or more readers from executing exit section. 
- `writer_wait` : If reader in critiacal section, waiting writer gets priority.  
- `allow` : Prevents reader from entering critical section while writer is writing and allows only one process at a time to excute the readers entry section.

**Variables Used:** 
-  DATA : Global shared data. 
- `waitwriter` : If writer is waiting when reader is in critial section its true. Else its false. 
- `popreader` : Total number of readers that have executed EXIT section. 
- `allowreader` : Total number of readers that have executed ENTRY section. 


