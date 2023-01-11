# Bisection Bandwidth Tester
This repo involves a unit test for measuring the bandwidth of communication of a group of processors (mainly GPUs). It is based on MPI but tests additional capabilites such as GPU-Aware MPI, CPU-Staged MPI, NCCL, and IPC.

Porting the capabilities are controlled by preprocessor directives. With no specification, it targets CPU by default. To port on Nvidia GPUs, one needs to ```#define SCI_CUDA```. To port on AMD GPUs, one needs to ```#define SCI_HIP```. Please refer to the table at the bottom to enable desired capabilities.

There are two parameters to describe the logical group topology. The first one is the number of processors and the second one is the group size. The benchmarking tool splits the global communicator ```MPI_COMM_WORLD``` into subcommunicators with ```MPI_Comm_split```. Eeach group talks to all other groups with a mapping between GPU as shown in the figure below. These partitioning scenarios can be applied to test communication bandwidth accross nodes, among GPUs within nodes, and between pairs of GPUs. In this scheme, each MPI rank runs a single GPU.

![Group Examples](https://github.com/merthidayetoglu/OLCF_BW_test/blob/main/scripts/group_examples.png)

Considering a hierarchical communication network, MPI ranks are assumed to be assigned in as an SMP style. For example, if there are six GPUs and three nodes, GPU 0 and GPU 1 are in the same node and so GPU 2 and GPU3, and so on. The first GPU of a group talks to the first GPUs on other groups, the second GPU of a group talks to the corresponding second GPUs on other groups, and so on.

This tool runs like
```cpp
mpirun -np #numproc Alltoall #count #numiter #groupsize
```
where count is the number of words between two GPUs.

Number of iterations is for averaging the bandwidth over many times. The program performs one warmup round and measures time over the number of iteration where each iteration are bulk-synchronized individually.

The figure below summarizes the Summit results. Note that all involved GPUs both sends and receives data and the measurement is given as aggregate bi-directional bandwidth of a group.

![Summit Measurement](https://github.com/merthidayetoglu/Bisection_Bandwidth_Tester/blob/main/scripts/summit_measurement.png)

This table summarizes the implemented capabilities.

| Porting Options   | Capability | Include |
| :---:               | ---: | :--- |
|Default is on CPU  | MPI | `#define MPI` |
|`#define SCI_CUDA` | CUDA-Aware MPI <br> CPU-Staged MPI <br> NCCL <br> CUDA IPC | `#define MPI` <br> `#define MPI_Staged` <br> `#define NCCL` <br> `#define IPC` |
|`#define SCI_HIP`  | GPU-Aware MPI <br> CPU-Staged MPI <br> (AMD port) NCCL <br> HIP IPC | `#define MPI` <br> `#define MPI_Staged` <br> `#define NCCL` <br> `#define IPC` |


Please send and email to [merth@stanford.edu](merth@stanford.edu) for any questions or contributions. Especially, extension of this benchmarking tool with GASNet-EX capability would be great!

Running on each system is like driving a different sports car, which has different handling and behaviour. This benchmarking tool helps understanding of the system characteristics. Our evaluation of various systems can be found below.

<details><summary>Summit Results</summary>
<p>

Results are provided in the figure above. Summit has IBM Spectrum MPI, which uses a lower-level API called parallel active message interfece (PAMI). There is a known problem with CUDA-Aware MPI and its way of allocating hardware buffers.

The PAMI variables are configured to have a lower latency [see here](https://docs.olcf.ornl.gov/systems/summit_user_guide.html#spectrum-mpi-tunings-needed-for-maximum-bandwidth). To obtain full theoretical bandwidth, we set up the PAMI variables as:
```bash
export PAMI_ENABLE_STRIPING=1
export PAMI_IBV_ADAPTER_AFFINITY=1
export PAMI_IBV_DEVICE_NAME="mlx5_0:1,mlx5_3:1"
export PAMI_IBV_DEVICE_NAME_1="mlx5_3:1,mlx5_0:1"
```
Thanks [Chris Zimmer](https://www.olcf.ornl.gov/directory/staff-member/christopher-zimmer/) for helping me understanding this!
</p>
</details>

<details><summary>Crusher Results</summary>
<p>

![Crusher Measurement](https://github.com/merthidayetoglu/Bisection_Bandwidth_Tester/blob/main/scripts/crusher_measurement.png)

</p>
</details>

<details><summary>Delta Results</summary>
<p>

</p>
</details>

<details><summary>Spock Results</summary>
<p>

</p>
</details>

<details><summary>ThetaGPU Results</summary>
<p>


</p>
</details>

For reproducibility, we provide the preworked Makefiles and run scripts for various systems, including Summit, Crusher, Spock, Delta, and ThetaGPU in the scripts folder.
