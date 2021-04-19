# implement of the parallel version
Suppose we will use 'size' processes in the program and there are 'num_values' elements in the stenci in total.
Root process(rank 0) is used to generate and deliver the elements. It also joins in computing. This parallel code will be in a manager/workers mode.
To parallelize the code, we need to deliver the elements to diffenet processes. Each process will get 'send_count' elements.
'send_count' is equal to num_values/size. 

To implement that, MPI_Scatter is used. This is a blocking function. The root process will deliver the same number of elements to all processes.
These elements will be stored in a local input array and the result of computing will be stored in a local output array.

To compute the stencil, we give the local input array and local output array extra 4 elements. 
2 are in the beginning of the array and another 2 are at the end of the array. The extra array elements are needed for the stencil application on neighboring nodes.
Each process will be given a left neighbor and a right neighbor via the following codes:
```C
    // Define left and right process
    int left, right;
    left = myid -1;
    if (left < 0){left = size -1;}
    right = myid + 1;
    if (right > size -1){right = 0;}
```
The first 2 extra elements come from the left neighbor. And the last 2 extra elements come from the right neighbor. 
Since the following computing is based on the data, we need to make sure all the data have been sent and received. Thus, we will use blocking 'send' and blocking 'receive' here.
To avoid 'deadlock', we choose to use MPI_Sendrecv. The code are as follows.
```C
MPI_Sendrecv(localinput+2, 2, MPI_DOUBLE, left, 1, localinput+send_count+2, 2, MPI_DOUBLE, right, 1, MPI_COMM_WORLD, &status);
MPI_Sendrecv(localinput+send_count, 2, MPI_DOUBLE, right, 0, localinput, 2, MPI_DOUBLE, left, 0, MPI_COMM_WORLD, &status);
```
The first MPI_Sendrecv means that, the current process will send 2 elements to its left process and receive 2 elements from its right process.
The elements sent are 'localinput+2' and 'localinput+3'. They will be stored in the last 2 position in the local input array of the left process.
The elements received will be stored in the last 2 position in the local input array of the current process.
The second MPI_Sendrecv means that, the current process will send 2 elements to its right process and receive 2 elements from its left process.
The elements sent are 'localinput+send_count' and 'localinput+send_count+1'. They will be stored in the first 2 position in the local input array of the right process.
The elements received will be stored in the first 2 position in the local input array of the current process.

After the communication is completed, the computation starts. Each process has send_count+4 elements. And the elements we need to update are from the third to the bottom third.
The computing of these elements which need to be updated is only based on the elements in the current process. Thus, the code is as follows:
```C
  // Apply stencil on points
	for (int i=2; i<send_count+2; i++) {
		double result = 0;
		for (int j=0; j<STENCIL_WIDTH; j++) {
			int index = i - EXTENT + j;
			result += STENCIL[j] * localinput[index];
		}
		localoutput[i] = result;
   }
```
To make sure all the process are in the same time step, a MPI_Barrier is set at the end of each time step.
Like the Scatter, Gather is used to collect the data from the other processes and store them in the root process.
```C
    //Gather data
    MPI_Gather(localoutput+2,send_count,MPI_DOUBLE,output,send_count,MPI_DOUBLE,0,MPI_COMM_WORLD);
```
A MPI_Reduce with MPI_Max is used to find the longest execution time.
```C
MPI_Reduce( &my_execution_time, &max_execution_time, 1, MPI_DOUBLE, MPI_MAX, 0, MPI_COMM_WORLD);
```

# measurement of time
When computing a stencil with 280000 values and 10 timesteps:

Serial code:  
![image](https://user-images.githubusercontent.com/65893273/115259878-db1f5f80-a164-11eb-86c0-26ca48a0df6b.png)  
Parallel code:  
![image](https://user-images.githubusercontent.com/65893273/115260104-0609b380-a165-11eb-940d-51b39219a5fb.png)  

The measurement of communication time and stencil time in one time step:

![image](https://user-images.githubusercontent.com/65893273/115260491-60a30f80-a165-11eb-8ca1-8f3cf4be3b90.png)  



