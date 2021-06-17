Evaluation of Legate after one month
====================================

## 1. Reproducing results of 12-step CFD
----------------------------------------

We first attempted to reproduce the throughput results from NVIDIA’s GTC presentation. By successfully reproducing the results, we would be more confident that we were correctly using Legate. A figure from the presentation shows a weak-scaling benchmark using the last step in the 12-step CFD and Legate with 1 to 2048 A100 GPUs. It evaluates the performance using the throughput – the number of grid points per second. The figure also shows the throughput of the same code using CuPy and only one A100 GPU.

Due to the limitation of the hardware resources, we tried to reproduce the throughput of one single A100 GPU rather than conduct the whole weak-scaling benchmark. However, we were not able to identify the simulation grid size from the figure in the presentation. Therefore, we ran the CFD simulation with several grid sizes and carried out a single-GPU throughput study for Legate and CuPy. While the result can reveal the throughput performance with respect to computational loading, the ultimate goal is to identify the grid size used by NVIDIA and confirm if the reproduction was successful.

The following figure shows the result of our throughput study. We calculated each data point using the mean value of 8 out of 10 identical runs. (We excluded the maximum and the minimum of the 10 identical runs.) Each run marched in time for 100 iterations with a time step size calculated from 0.95 maximum CFL number. The throughput calculation includes boundary grid points.

![cfd_throughput_single_a100.png](figures/cfd_throughput_single_a100.png)

In NVIDIA’s figure, when using one A100, CuPy gives a throughput of about 1.66e8 points/second, and Legate gives 1.54e8 points/second. The ratio is about 1.08. From our benchmark, we were not able to obtain similar throughput values from all grid sizes. However, we obtained a throughput ratio of about 1.07 at about 1.6e8 grid points. The throughput values are 1.45e8 and 1.36e8 points/second for CuPy and Legate, respectively, at this grid size. Among all the grid sizes we tested, this grid size is also the maximum size that Legate can handle with one A100. We believe NVIDIA used a grid size close to or slightly greater than it.

Though the absolute throughput values do not match, we consider the reproduction to be successful. Many factors may contribute to the difference in the absolute throughput. For example, NVIDIA did not detail how they calculated the throughput. Did they include the boundary points? How many temporal iterations did they use? The difference in how we instrumented the code also affects the absolute values.

## 2. User experience
---------------------

### 2.1 Installation
--------------------

Legate is still a work-in-progress in its early development stage, so we think the current installation process is good enough. However, one thing we noticed is the use of hidden JSON files for recording previous installation information. For many Python users, using virtual environments (e.g., conda environments, built-in `venv`) are common. Users likely want to install Legate into several different environments. If users are unaware of the hidden JSON files, an installation of Legate in an environment may be linked to a third-party library in another environment.

### 2.2 Documentation and usage
-------------------------------

Legate requires users to launch Python programs using an executable -- `legate`. While `legate --help` shows available command-line arguments and brief explanations of the arguments, some arguments are still unclear to users, especially those who have zero knowledge of the Legion runtime system. We often have to look into the source code or go to Legion’s documentation to understand what an argument does or how to use it. An example is [issue 4](https://github.com/nv-legate/legate.core/issues/4) at the Legate Core repository. This issue clarifies the configuration of OpenMP processes, threads, CPU processes, and utility processes. We were not able to find this information in either Legate's or Legion's documentation.

On the other hand, in the same issue, developers brought out more command-line arguments from the underlying Legion runtime system: `-ll:okindhack` and `-ll:ht_sharing`. We could not find the information of these arguments in even Legion's documentation.

Another impression of Legate's usage is the command-line argument parsing system. As shown in our reply to [issue 6](https://github.com/nv-legate/legate.numpy/issues/6) at Legate NumPy repository, when mixing the arguments from Legate, Legion, and an application, the order of the arguments matters. How to determine the order, however, is unclear to users. The comments in [issue 2](https://github.com/nv-legate/legate.core/issues/2) have more discussions about Legate's argument parsing system. [Issue 3](https://github.com/nv-legate/legate.core/issues/3) and [16](https://github.com/nv-legate/legate.core/issues/16) at Legate Core and [issue 3](https://github.com/nv-legate/legate.numpy/issues/3) at Legate NumPy are other command-line argument issues we encountered during installation and using Legate.

Though most of these argument-related issues can be quickly fixed or not a current focus at the current development stage, they give an impression that Legate's argument parsing system probably still needs some improvement for average Python users to use.
