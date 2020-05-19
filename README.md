# OpenMP Offload/Blas Examples
Example C and Fortran code showing how to offload blas calls from OpenMP regions,
using cuBLAS and NVBLAS.

There are two directories: 
 - cublas
 - nvblas

These contain Makefiles and examples of calling DGEMM from an OpenMP
offload region with cuBLAS and NVBLAS.

To run them, cd into eitheer cublas or nvblas, then cd into a "c" or "fortran" subdirectory,
and compile with make.

`dgemm_nvblas` uses NVBLAS, and `dgemm_cublas` explicitly uses cuBLAS
`dgemm_cublas_fortran` calls cuBLAS from Fortran using iso_c bindings.

A snippet of the code, showing the interface is:

For cublas:
```
#pragma omp target data map(to:aa[0:SIZE*SIZE],bb[0:SIZE*SIZE],alpha,beta) map(tofrom:cc_gpu[0:SIZE*SIZE]) use_device_ptr(aa,bb,cc_gpu)
 {
   cublasDgemm(handle,CUBLAS_OP_N, CUBLAS_OP_N,SIZE, SIZE, SIZE, &alpha, aa, SIZE, bb, SIZE, &beta, cc_gpu, SIZE);
  }
```

For nvblas:
```
#pragma omp target data map(to:aa[0:SIZE*SIZE],bb[0:SIZE*SIZE],alpha,beta) map(tofrom:cc_gpu[0:SIZE*SIZE]) use_device_ptr(aa,bb,cc_gpu)
 {
   dgemm("N","N",&size, &size, &size, &alpha, aa, &size, bb, &size, &beta, cc_gpu, &size);
  }
```

Makefile is hardcoded to run on Summit.

# Building and Running on Summit

 To run, the simplest is to submit an interactive job:

 1. Submit an interactive job:
    ```
    $ bsub -W 1:00 -nnodes 1 -P Project -Is $SHELL
    ```
 2. Set the environment:
    ```
    $ source environment_setup.sh
    ```
    This loads the cuda module, which is needed for tests
    with nvprof

 3. Move into a subdirectory (e.g. nvblas/c or cublas/fortran)
    ```
    $ cd cublas/fortran
    ``` 

 3. Compile the file
    ```
    $ make 
    ```

 4. Use jsun to run each simple example:
    ```
    $ jsrun -n 1 -a 1 -c 1 -g 1 ./dgemm_nvblas

       -n is the nubmer of resource sets,
       -a is the number of MPI ranks,
       -c is the number of physical cores,
       -g is the number of GPUs

    $ jsrun -n 1 -a 1 -c 1 -g 1 nvprof --print-gpu-trace ./dgemm_cublas
    ```
     etc.
