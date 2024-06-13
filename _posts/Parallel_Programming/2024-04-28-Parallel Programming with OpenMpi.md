---
layout: post
title: Parallel programming with OpenMpi
subtitle: 
gh-repo: honky-tonk/honky-tonk.github.io
gh-badge: [star,follow]
tags: [Parallel Programming]
comments: true
---
# Preface
# MPI Basics

There are six basics function in MPI

> ```MPI_Init```: initialize mpi

>```MPI_Comm_size```: Find out how many processes there are

>```MPI_Comm_rank```: Find out which process i am

>```MPI_Send```: send a message

>```MPI_Recv```: receiven a message

>```MPI_Finalize```: Terminate MPI

Other function in MPI

> ```MPI_Comm_rank(COMMNICATOR, &rank)```: get current rank in this Communicator

> ```MPI_Comm_size(COMMNICATOR, &size)```: get rank size of this Communicator

So, what is rank and Communicator? apparently a Communicator contains multiple rank, a rank can be considerd as a process than launch in a node and each rank has union id in same Communicator, a communicator can be considerd as Container, ```MPI_COMM_WORLD``` is a predefined communicator that represents the entire MPI universe, you can use ```MPI_Comm_split()```,```MPI_Comm_group()``` and other function to **SPLIT** ```MPI_COMM_WORLD``` communicator, but Rank id may **NOT** union with inter-communicator , because each rank id is assigned when a process join the communicator. 

# Ping-Pong Example

```cpp
/*ping-pong.c*/
#include <mpi.h>

int main(int argc, char **argv) {
  MPI_Init(&argc, &argv);

  int rank, size;
  MPI_Comm_rank(MPI_COMM_WORLD, &rank);
  MPI_Comm_size(MPI_COMM_WORLD, &size);

  if (size != 50) {
    printf("Error: This program requires 50 MPI processes.\n");
    MPI_Finalize();
    return 1;
  }

  int partner_rank = (rank + 1) % size; // Determine partner rank

  // Ping-Pong communication
  int message;
  for (int i = 0; i < 2; i++) {
    if (rank % 2 == 0) { // Even ranks send first
      message = rank;
      MPI_Send(&message, 1, MPI_INT, partner_rank, 0, MPI_COMM_WORLD);

      MPI_Recv(&message, 1, MPI_INT, partner_rank, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
      printf("Received message from rank %d: %d\n", partner_rank, message);
    } else { // Odd ranks send second
      MPI_Recv(&message, 1, MPI_INT, partner_rank, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
      printf("Received message from rank %d: %d\n", partner_rank, message);

      message++;
      MPI_Send(&message, 1, MPI_INT, partner_rank, 0, MPI_COMM_WORLD);
    }
  }

  MPI_Finalize();

  return 0;
}
```
Then we compile&&run the script
```
mpicc -o ping_pong ping-pong.c
mpirun -np 50 ping-pong
``` 
first we use 50 rank to run this program(```-np 50```), In the front few line, we used ```MPI_Comm_rank``` and ```MPI_Comm_size``` to get the Current rank(process) id, and size of the Communicator(how many rank the commnunicator have), then we use even rank to ping, and next odd rank to pong


# Syncsend and Buffersend
**Syncsend**: sender send message to reciver, when reciver recive, sender send could be done. we can use ```MPI_Ssend()``` and ```MPI_Recv()``` to send and recive message

**Buffersend**: sender send message to reciver's buffer, when buffer is filled, reciver return immediately. we can use ```MPI_Bsend()``` and ```MPI_Recv()``` to send and recive message

---

**Warning**
Consider below code
```
if(rank == 0){
  MPI_Ssend(&val1, 1, MPI_INT, 1, tag1, MPI_COMM_WORLD);
  MPI_Ssend(&val2, 1, MPI_INT, 1, tag2, MPI_COMM_WORLD);
}else if(rank == 1){
  MPI_Recv(&val2, 1, MPI_INT, 0, tag2, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
  MPI_Recv(&val1, 1, MPI_INT, 0, tag1, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
}
```
first, take look at rank 1, when exec ```MPI_Recv(&val2, 1, MPI_INT, 0, tag2, MPI_COMM_WORLD, MPI_STATUS_IGNORE);``` rank 1 will get blocked until rank 0 send message which tag is tag 2, but rank 0 get blocked, when exec first line ```MPI_Ssend(&val1, 1, MPI_INT, 1, tag1, MPI_COMM_WORLD);```, because rank 1 in block and not able to recive message which tag is tag1.apparently, this is **dead lock**!

To address this issue, all we need to do is change the sequence of rank 1 code like below
```
if(rank == 0){
  MPI_Ssend(&val1, 1, MPI_INT, 1, tag1, MPI_COMM_WORLD);
  MPI_Ssend(&val2, 1, MPI_INT, 1, tag2, MPI_COMM_WORLD);
}else if(rank == 1){
  MPI_Recv(&val1, 1, MPI_INT, 0, tag1, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
  MPI_Recv(&val2, 1, MPI_INT, 0, tag2, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
}
```
