# DAWN V2.1

DAWN is a novel shortest paths algorithm, which is suitable for weighted and unweighted graphs. DAWN requires $O(m)$ space and $O(S_{wcc} \cdot E_{wcc})$ times on the unweighted graphs, which can also process SSSP tasks and requires $O(E_{wcc}(i))$ time. $S_{wcc}$ and $E_{wcc}$ denote the number of nodes and edges included in the largest WCC (Weakly Connected Component) in the graphs.

DAWN is capable of solving the APSP and SSSP problems on graphs with negative weights, and can automatically exclude the influence of negative weight cycles.  

## Quick Start Guide

### 0. Before getting started

Depending on your GPU, you may also want to edit the CMAKE_CUDA_ARCHITECTURES variable in $PROJECT_ROOT/CMakeLists.txt

```c++
export PROJECT_ROOT="to_your_project_path"
```

### 1. Modify $PROJECT_ROOT/CMakeLists.txt

According to your GPU, we use RTX 3080ti for computing, so CMAKE_CUDA_ARCHITECTURES is set to 86

```c++
set(CMAKE_CUDA_ARCHITECTURES "xx")
set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -O3 -gencode arch=compute_xx,code=sm_xx")
```

Set the CMAKE_CUDA_COMPILER to the path of your NVCC, for example,

```c++
set(CMAKE_CUDA_COMPILER "/usr/local/cuda/bin/nvcc")
```

### 2.Download testing data

Unzip the compressed package and put it in the directory you need

The input data can be found on the Science Data Bank

```c++
URL=https://www.scidb.cn/s/6BjM3a
GRAPH_DIR="to_your_graph_path"
```

### 3. Dependencies

DAWN builds, runs, and has been tested on Ubuntu/Rocky Linux. Even though DAWN may work on other linux systems, we have not tested correctness or performance. DAWN is not available on Windows and cannot achieve the same performance on WSL systems. Please beware.

At the minimum, DAWN depends on the following software:

- A modern C++ compiler compliant with the C++ 14 standard
- GCC (>= 9.4.0 or Clang >= 10.0.0)
- CMake (>= 3.10)
- libomp (>= 10.0)

If you need run DAWN on the GPU, expand:

- CUDA (>= 11.0)
- thrust (>= 2.0)

### 4. Build and RUN

```c++
cd $PROJECT_ROOT
mkdir build && cd build
cmake .. && make -j
```

If compilation succeeds without errors, you can run your code as before, for example

```c++
cd $PROJECT_ROOT/build
./dawn_cpu_apsp SG $GRAPH_DIR/mouse_gene.mtx ../output.txt 100 false 0 unweighted

./dawn_cpu_apsp SG $GRAPH_DIR/cage10.mtx ../output.txt 100 false 0 weighted

./dawn_gpu_apsp Default $GRAPH_DIR/mouse_gene.mtx ../output.txt 4 256 100 false 0 unweighted

./dawn_gpu_apsp Default $GRAPH_DIR/cage10.mtx ../output.txt 4 256 100 false 0 weighted
```

When the version is built, it will also prepare SSSP and MSSP applications, which can be used directly.

If you need to use DAWN in your own solution, please check the source code under the **sssp**, **mssp** folder and call them.

If you do not have the conditions to use NVCC, you can use the **cmakelistsforcpu.txt** (It should be renamed before building), then use GCC or clang to build applications that can only run on the cpu. (GCC 9.4.0 and above, clang 10.0.0 and above).

#### Using script

```c++
cd ..
vim ./process.sh
MAIN = ${main}
GRAPH_DIR = ${test_graph}
OUTPUT= ${outputfile}
LOG_DIR= ${GRAPH_DIR}/log
ESC && wq
sudo chmod +x ../process.sh
bash ../process.sh
```

Please note that the normal operation of the batch script needs to ensure that the test machine meets the minimum requirements. Insufficient memory or GPU memory needs to be manually adjusted according to amount of resources.

#### For general graphs

```c++
CPU: Multi-threaded processor supporting OpenMP API
RAM: 8GB or more
GPU: 1GB or more
Compiler: NVCC of CUDA 11.0 above
OS:  Ubuntu 20.04 and above
```

#### For large-scale graphs

```c++
CPU: Multi-threaded processor supporting OpenMP API
RAM: 24GB or more
GPU: 4GB or more
Compiler: NVCC of CUDA 11.0 above
OS:  Ubuntu 20.04 and above
```

### 5.Release version

| Version | Implementation |
| ------ | ------ |
| APSP/TG |  Version of Thread Parallel|
| APSP/SG |  Version of Stream Parallel|
| MSSP/S | Version of Multi-source via Thread Parallel|
| MSSP/P | Version of Multi-source via Stream Parallel|
| SSSP | Version of Single-source|

Currently, on the Single-Processor, SG allocates one thread per stream. On Multi-Processor, SG will allocate multiple threads per stream. (It will be supported in the next version.)

For the GPU version, you can use Default, please make sure that there is enough GPU memory for the graph. If you are sure that the size of the thread block and the scale of the graph is set reasonably according to the device parameters.

```c++
int device;
cudaDeviceProp props;
cudaGetDevice(&device);
cudaGetDeviceProperties(&props, device);
printf("Max shared memory per block: %ld\n", props.sharedMemPerBlock);
```

### 6.Release result

On Intel Core i5-13600KF, if you use **SG**, it takes about **15** minutes to complete the test on 36 graphs. On AMD EPYC 7T83, if you use **SG**, it takes about **5** minutes to complete the test. The test time mentioned here refers to the running time of the program and does not include the data read-in and output time.

On the test machine with i5-13600KF, achieves an average speedup of 89.030&times; and 96.152&times; over BFS and GDS (hereinafter referred to as GDS and BFS), respectively. On the test machine with i5-13600KF, DAWN need more than 10GB free memory to solve the large graph [281K,214M].

On the 7T83 processor, DAWN based on SOVM exhibits high scalability, achieving an average speedup of 3.487&times;, 335.325&times; and 310.488&times; over GDS, BFS and DAWN(20), respectively.

On the test machine with RTX3080TI, DAWN achieves average speedup of 313.763&times;, 338.862&times; and 3.524&times;, over GDS, BFS and DAWN(20), respectively.

We provide the file **check_unweighted.py** and **check_weighted.py**, based on networkx, which can be used to check the results printed by DAWN.

### 7.Documentation

Please refer to [document/Documentation_v1](https://github.com/lxrzlyr/SC2023/blob/eb9080f76c2950981a4dac72141d4991eff8b9db/document/Decumentation_v1.md) for commands.

## New version

The version of DWAN on the weighted graph has been included in DAWN V2.1. Currently, DAWN includes the version that runs on unweighted graphs of int type index values, and the version that runs on negative weighted graphs of float type. (SOVM and GOVM have been the default implementation, if you want to use BOVM, please change the kernel function.)

In the future, we plan to develop more algorithms based on DAWN, including but not limited to Between Centrality, Closeness Centrality, etc. Further applications of these algorithms, such as community detection, clustering, and path planning, are also on our agenda.

We welcome any interest and ideas related to DAWN and its applications. If you are interested in DAWN algorithms and their applications, please feel free to share your thoughts via [email](<1289539524@qq.com>), and we will do our best to assist you in your research based on DAWN.

 The DAWN component based on Gunrock may be released to the main/develop branch in the near future, so please stay tuned to the [Gunrock](https://github.com/gunrock/gunrock). We will release new features of DAWN and the application algorithms based on DAWN on this repository. If the algorithms are also needed by Gunrock, we will contribute them to the Gunrock repository later.

<!-- ## How to Cite DAWN

Thank you for citing our work.

```bibtex
@misc{feng2023novel,
      title={A Novel Shortest Paths Algorithm on Unweighted Graphs}, 
      author={Yelai Feng and Huaixi Wang and Yining Zhu and Chao Chang and Hongyi Lu},
      year={2023},
      eprint={2208.04514},
      archivePrefix={arXiv},
      primaryClass={cs.DC}
}
@misc{feng2023expanding,
      title={Expanding the Scope of DAWN: A Novel Version for Weighted Shortest Path Problem}, 
      author={Yelai Feng},
      year={2023},
      eprint={2306.07872},
      archivePrefix={arXiv},
      primaryClass={cs.DS}
}
``` -->

## Copyright & License

All source code are released under [Apache 2.0](https://github.com/lxrzlyr/DAWN-An-Noval-SSSP-APSP-Algorithm/blob/4266d98053678ce76e34be64477ac2364f0f4291/LICENSE).
