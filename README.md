[![DOI](https://zenodo.org/badge/516906791.svg)](https://zenodo.org/badge/latestdoi/516906791)
<img src="https://sysartifacts.github.io/images/acm_available_1.1.png" alt="available" width="40" height="40">
<img src="https://sysartifacts.github.io/images/acm_functional_1.1.png" alt="functional" width="40" height="40">
<img src="https://sysartifacts.github.io/images/acm_reproduced_1.1.png" alt="reproduced" width="40" height="40">

# STRONGHOLD: Fast and Affordable Billion-scale Deep Learning Model Training

Deep neural networks (DNNs) with billion-scale parameters have demonstrated impressive performance in solving many tasks. Unfortunately, training a billion-scale DNN requires high-performance GPU servers that are too expensive to purchase and maintain. Existing solutions for enabling larger DNN training with limited resources are inadequate because they suffer from high training time overhead.

We present StrongHold, a better approach for enabling large DNN model training by dynamically offloading data to the CPU RAM and using the secondary storage (e.g., an SSD drive). It maintains a working window to overlap the GPU computation with CPU-GPU data movement carefully and exploits the multi-core CPU for optimizer update. Compared to the state-of-the-art offloading-based solutions, StrongHold improves the trainable model size by 1.9x∼6.5x on a 32GB V100 GPU, with 1.2x∼3.7x improvement on the training throughput.

## Our experiment setup

#### Hardware
NVIDIA Tesla V100-GPU server with 2x 24-core Intel Xeon Platinum 8163 CPU at 2.50GHz and 755GB of DDR4 RAM. NVIDIA GPU (arch=sm_70) with device memory is 32GB. It is possible to evaluate our approach on a different NVIDIA GPU platform, but the results may deviate from the ones reported in the paper.  

Note that for the Artifact Evaluation, we have rent a VM with one V100 for reviewers.

#### Operating System
Ubutun 20.04 with Linux kernel 4.19.91

#### Software
CUDA 11.6, PyTorch 1.10.2

## How to reproduce the experiment in the submitted paper?
####  Choice-1: An interactive jupyter notebook.
We have provided an interactive jupyter notebook run in a preconfigured server with a 32GB V100 GPU to make the reviewing process more straightforward. The runtime environment has been set, and all the scripts together with their descriptions have also been well-prepared for reviewers to reproduce the main results of the paper.  <!-- We recommend this choice to reviewers since the testing cases, launch scripts and descriptions have also been listed in this notebook. -->   <br>
**jupyter link**:  [http://47.111.26.83:8888/notebooks/sc22ae.ipynb](http://47.111.26.83:8888/notebooks/sc22ae.ipynb) <br>
**password**: see the ` Stage 1: Artifact Description` in the [sc submission system](https://submissions.supercomputing.org/) <br>

####  Choice-2: The docker image.
Reviewers can also deploy the runtime environment via this docker image if they have available GPU(s) resources.  <br>
Please note to change the values of script arguments and re-install the torch if the CUDA driver version is lower than 11.4, also highlighted in the next content. <br>
The following content describes the steps on how to use this docker image to reproduce experiments.

----

# StrongHold: Artifact Evaluation

This docker image helps reviewers reproduce the experiment results in the submitted paper for the AE committee. It has already included all the necessary datasets and baselines. Thus, following the instructions, most figures in the paper can be reproduced.

We provide a general description of how to launch a docker container using this image and how to run corresponding scripts to evaluate the system performance.

## Prerequirement

Before getting started, please make sure a Docker Engine has been installed. We recommend the installation of [Docker Community Edition (CE)](https://docs.docker.com/get-docker/). 

Or, a Linux user can run the following commands in a Linux terminal to get a Docker:
```bash
sudo apt-get update
sudo apt-get install -y docker.io
```

Running the Docker command requires root privileges. It can be run by a user without root privileges by adding the username to the Docker group:
```bash
sudo usermod -a -G docker $USER
```

Run the following commands to enable GPU in Docker environment with NVIDIA container toolkit on the host machine. Details are from https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker.

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
  && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

Then, install the nvidia-docker2 package after updating the package listing:

```bash
sudo apt-get update 
sudo apt-get install -y nvidia-docker2
sudo systemctl restart docker
```

Next, test if the GPU running environment is successfully configured. A working setup can show the GPU information by running a base CUDA container:

```bash
sudo docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```

The Docker and the Host GPU should all be ready if the above settings do not go south.

## Step 1. Runtime Environment

### 1.1 Download this docker image from the official hub website.

```bash
docker pull strongh/sc22-ae:latest
```

### 1.2  Create a new docker container based on this image. 

```bash
docker run -it -P -w /home/sys/STRONGHOLD --name=aetesting --network=host --gpus=all --ipc=host strongh/sc22-ae:latest /bin/bash`
```

### 1.3 Check the runtime environment.

At this point, I believe the docker container is launched successfully, and the current terminal focus should be at the `/home/sys/STRONGHOLD` folder with `(py3.9.10)` virtual python environment, shown as

 `(py3.9.10) root@??:/home/sys/STRONGHOLD#`

If not at the `STRONGHOLD` folder, please use the `cd /home/sys/STRONGHOLD` command to change the current location.

If not at `(py3.9.10)` virtual python environment, please use `pyenv activate py3.9.10` to enter into `py3.9.10` virtual environment.

#### Note: if the host server's CUDA driver is lower than 11.4 or errors about the torch version occur, please reinstall `torch`, `torchvision` and `apex` libraries using the following commands.
>
```bash
pip uninstall torch torchvision apex

STAGE_DIR=/home/sys/
cd ${STAGE_DIR}/pytorch && \
    pip install -r requirements.txt && \
    python setup.py clean && \
    python setup.py develop --cmake && \
    python setup.py install && \
cd ${STAGE_DIR}/vision && \
    python setup.py clean && \
    python setup.py install && \
cd ${STAGE_DIR}/apex && \
    pip install -r ./requirements_dev.txt && \
    pip install -v --no-cache-dir --global-option='--cpp_ext' --global-option='--cuda_ext' . && \
cd ${STAGE_DIR}/STRONGHOLD
```

### 1.4 Experiment workflow
The evaluation script is located at the `examples` folder, which should be executed from the parent directory. The script contains a number of configurable arguments to change the DNN setup. The script itself should self explain these parameters, which are summarized as follow: 

- number of layers, (default=16)
- hidden size, (default=2048)
- number of heads, (default=16)
- sequence length, (default=1024)
- batch size, (default=4)
- window size, (default=4) (only for StrongHold)

The script can be executed by:
```bash
./examples/run.sh -m [METHOD] -l [NUM_LAYERS] -h [HIDDEN_SIZE] -b [BATCH_SIZE] -w [WINDOW_SIZE]
```
where the [METHOD] argument takes the following values: `megatron-lm`, `l2l`, `zero-offload`, `zero-infinity`, `stronghold` and `all`. Using `all` to automatically evaluate all approaches; the `[NUM_LAYERS]`, `[HIDDEN_SIZE]`, `[BATCH_SIZE]` and `[WINDOW_SIZE]` take integers, indicating the number of transformer layers, the hidden size of each layer, the training batch size and offloading window size, separately.

#### Results:
Evaluation results will be saved into the `results` folder, formatted as `log_[METHOD]_l-[NUM_LAYERS]_h-[HIDDEN_SIZE]_bs-[BATCH_SIZE]_ws-[WINDOW_SIZE]_[TIMESTAMP].txt`. The scripts of `./examples/case*_extract.sh` and `./examples/case*_draw.py` produce auto-generated diagrams, where `*` means the testing case id.

#### Logs:
Log files are stored in `/home/sys/STRONGHOLD/results` as a format of `log_[METHOD]_l-[NUM_LAYERS]_h-[HIDDEN_SIZE]_bs-[BATCH_SIZE]_ws-[WINDOW_SIZE]_[TIMESTAMP].txt`. We print the core content in the log files via `grep` and `awk` for you at the end of each execution. <!-- You can also view the additional progress of the script by running: `tail -f results/log-[METHOD]-[TIMESTAMP].txt` -->

#### The next step will give more details to help you reproduce the major results in our paper. Please follow the guidelines below to evaluate each case.

## Step 2. Evaluation.

To reuse the existing log files produced in the previous cases, we recommend running the following cases one by one, which reduces the total execution time to **around 5 hours**. 

- 2.1 CASE - The largest trainable model size (Figure 6a in Section VI.A)
- 2.2 CASE - Throughput  on the largest trainable model size supported by each baseline (Figure 7a in Section VI.B) 
- 2.3 CASE - Throughput on the largest trainable model size of Megatron-LM (Figure 8a in Section VI.B)
- 2.4 CASE - Nearly linear scaling as model size increases (Figure 8b in Section VI.B) 
- 2.5 CASE - Impact of working window size (Figure 9 in Section VI.C) 

**Log files are stored in `/home/sys/STRONGHOLD/results`** as a format of `log_[method]_l-[layers]_h-[hidden size]_bs-[BATCH_SIZE]_ws-[WINDOW_SIZE]_[date].txt`. We print the core content in the log files via `grep` and `awk` for you at the end of each execution.

**Launch script** `./examples/run.sh -m [method] -l [layers] -h [hidden size] -b [batch size] -w [window size]` accepts five arguements, where `[method]` takes the values of `megatron-lm`, `l2l`, `zero-offload`, `zero-infinity`, `stronghold` and `all`. Using all to automatically evaluate all approaches. Default values for `[layers]`, `[hidden size]`, `[batch size]`, `[window size]` are 16, 2048, 4 and 4, respectively.

### 2.1 The evaluation example on virtual machine with a single 32GB V100 GPU and a CPU (with 90GB RAM and 12 CPU Cores) .
> **!!! Please change the corresponding arguments if the hardware differs from ours. !!!**

#### 2.1.1 CASE - The largest trainable model size (Figure 6a in Section VI.A)

In this case, we use GPT-like models to exploit each method's largest trainable model size. Model size changes via increasing/decreasing the number of transformer layers.

We evaluate Megatron-LM, L2L, ZeRO-Offload, ZeRO-Infinity and STRONGHOLD on a virtual machine with a single 32GB V100 GPU and a CPU (with 90GB RAM and 12 CPU Cores) to exploit their largest trainable model size and bottleneck. During this process, we set the `Heads=16, Sequence Length=1024, Batch Size=4` in all GPT-like models and training setups.

The largest model sizes have been tested and shown in the following table. 

| Methods | Largest Trainable Size | Layers | Hidden Size | Heads | Sequence Length | Batch Size |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Megatron-LM | **1.717 B**| **32** | 2048 | 16 | 1024 | 4 |
| L2L | **4.033 B**| **78** | 2048 | 16 | 1024 | 4 |
| ZeRO-Offload | **2.522 B**| **48** | 2048 | 16 | 1024 | 4 |
| ZeRO-Infinity | **2.522 B**| **48** | 2048 | 16 | 1024 | 4 |
| STRONGHOLD | **5.141 B**| **100** | 2048 | 16 | 1024 | 4 |

To reproduce the results above, you can either turn to our provided notebook and run the corresponding cell (Cell Heading: **2.1 CASE - The largest trainable model size (Figure 6a in Section VI.A)**) or run the following commands within our docker container.

PS: `Errors about GPU/CPU OOM` might be represented as other information, such as 'can not create XXX'.

```bash
./examples/run.sh -m "megatron-lm" -l 32 -h 2048 && \
./examples/run.sh -m "l2l" -l 78 -h 2048 && \
./examples/run.sh -m "zero-offload" -l 48 -h 2048 && \
./examples/run.sh -m "zero-infinity" -l 48 -h 2048 && \
./examples/run.sh -m "stronghold" -l 100 -h 2048
```

> **`./examples/case1_extract.sh` and `./examples/case1_draw.sh` help you analysis log files and print the relevant information only.**


#### 2.1.2 CASE - Throughput on the largest trainable model size supported by each baseline (Figure 7a in Section VI.B) 

In this case, we use GPT-like models to exploit the largest trainable model size supported by each baseline and compare the performance against STRONGHOLD on each largest model size. Model size changes via increasing/decreasing the number of transformer layers.

We evaluate (Megatron-LM, L2L, ZeRO-Offload, ZeRO-Infinity) v.s. STRONGHOLD on a virtual machine with a single 32GB V100 GPU and a CPU (with 90GB RAM and 12 CPU Cores). During this process, we set the `Heads=16, Sequence Length=1024, Batch Size=4` in all GPT-like models and training setups.

The throughput has been tested and shown in the following table.

| Methods | Throughput | Trainable Size | Layers | Hidden Size | Heads | Sequence Length | Batch Size |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Megatron-LM | **0.7496** |1.717 B | 32 | 2048 | 16 | 1024 | 4 |
| STRONGHOLD | **0.6647** | 1.717 B| 32 | 2048 | 16 | 1024 | 4 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| L2L | **0.0529** | 4.033 B| 78 | 2048 | 16 | 1024 | 4 |
| STRONGHOLD | **0.2271** | 4.033 B| 78 | 2048 | 16 | 1024 | 4 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| ZeRO-Offload | **0.2523** |2.522 B | 48 | 2048 | 16 | 1024 | 4 |
| STRONGHOLD | **0.3999**| 2.522 B| 48 | 2048 | 16 | 1024 | 4 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| ZeRO-Infinity | **0.2439** | 2.522 B| 48 | 2048 | 16 | 1024 | 4 |
| STRONGHOLD | **0.3999**| 2.522 B| 48 | 2048 | 16 | 1024 | 4 |

To reproduce the results above, you can either turn to our provided notebook (Cell Heading: **2.2 CASE - Throughput on the largest trainable model size supported by each baseline (Figure 7a in Section VI.B)**) and run the corresponding cell or run the following commands within our docker container.

PS: Limitations of CPU cores and bandwidth in the virtual machine hurts the performance of STRONGHOLD a little.

```
./examples/run.sh -m "stronghold" -l 32 -h 2048 -w 15 && \
./examples/run.sh -m "stronghold" -l 48 -h 2048 -w 15 && \
./examples/run.sh -m "stronghold" -l 78 -h 2048 -w 15
```

> **`./examples/case2_extract.sh ` and `./examples/case2_draw.sh ` help you analysis log files and print the relevant information only.**


#### 2.1.3 CASE - Throughput on the largest trainable model size of Megatron-LM (Figure 8a in Section VI.B) 

This case shows the throughput performance of running Megatron-LM, L2L, ZeRO-Offload, ZeRO-Infinity and STRONGHOLD, respectively, on a 1.717 B model that is the largest trainable model size supported by Megatron-LM. The evaluation is conducted on a virtual machine with one 32GB V100, 90GB CPU RAM and 12 CPU Cores.

The throughput results have been tested and shown in the following table.

| Methods | Throughput | Trainable Size | Layers | Hidden Size | Heads | Sequence Length | Batch Size |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Megatron-LM | **0.7496** | 1.717 B| 32 | 2048 | 16 | 1024 | 4 |
| L2L | **0.1729**| 1.717 B| 32 | 2048 | 16 | 1024 | 4 |
| ZeRO-Offload | **0.3711**| 1.717 B| 32 | 2048 | 16 | 1024 | 4 |
| ZeRO-Infinity | **0.3587** | 1.717 B| 32 | 2048 | 16 | 1024 | 4 |
| STRONGHOLD | **0.6647** | 1.717 B| 32 | 2048 | 16 | 1024 | 4 |


To reproduce the results above, you can either turn to our provided notebook and run the corresponding cell (Cell Heading: **2.3 CASE - Throughput on the largest trainable model size of Megatron-LM (Figure 8a in Section VI.B)**) or run the following commands within our docker container.

PS: Limitations of CPU cores and bandwidth in the virtual machine hurts the performance of STRONGHOLD a little.

```
./examples/run.sh -m "l2l" -l 32 -h 2048 && \
./examples/run.sh -m "zero-offload" -l 32 -h 2048 && \
./examples/run.sh -m "zero-infinity" -l 32 -h 2048
```

> **`./examples/case3_extract.sh ` and `./examples/case3_draw.sh ` help you analysis log files and print the relevant information only.**


#### 2.1.4 CASE - Nearly linear scaling as model size increases (Figure 8b in Section VI.B)

In this case, we evaluate the performance (elapsed time per iteration - ms) as the model size increases. Similar to previous cases, the model size changes via increasing/decreasing the number of transformer layers. 

You would see the `elapsed time per iteration` linearly rise with the number of transformer layers (representing model size), proving STRONGHOLD's scalability.

Similar to the previous cases, you can reproduce the results in our notebook (the corresponding Cell Heading: **2.4 CASE - Nearly linear scaling as model size increases (Figure 8b in Section VI.B)**) or run the following commands in our docker container.

```
./examples/run.sh -m "stronghold" -l 92 -h 2048 -w 15 && \
./examples/run.sh -m "stronghold" -l 64 -h 2048 -w 15 && \
./examples/run.sh -m "stronghold" -l 56 -h 2048 -w 15 && \
./examples/run.sh -m "stronghold" -l 40 -h 2048 -w 15 && \
./examples/run.sh -m "stronghold" -l 24 -h 2048 -w 15 && \
./examples/run.sh -m "stronghold" -l 16 -h 2048 -w 15
```

> **`./examples/case4_extract.sh ` and `./examples/case4_draw.sh ` help you analysis log files and print the relevant information only.**


#### 2.1.5 CASE - Impact of working window size (Figure 9 in Section VI.C) 

Working window size affects the throughput. A larger window can better overlap GPU computation with data transfer, leading to higher training throughput. However, a larger window size means more GPU memory occupancy.

This case evaluates the impact of working window size for STRONGHOLD with 1.7B model. You will see that at the first stage, the larger window size can gain more benefits, while at the end of the stage, enlarging window size shows no influence because the current window size can hide the data transformation process.

To reproduce this, you can either turn to our notebook (the corresponding Cell Heading: **2.5 CASE - Impact of working window size (Figure 9 in Section VI.C)**) or run the following commands in our docker container:

PS: The bandwidth restriction in the virtual machine might slightly hurt the performance of STRONGHOLD.

```
./examples/run.sh -m "stronghold" -l 32 -h 2048 -w 2 && \
./examples/run.sh -m "stronghold" -l 32 -h 2048 -w 4 && \
./examples/run.sh -m "stronghold" -l 32 -h 2048 -w 6 && \
./examples/run.sh -m "stronghold" -l 32 -h 2048 -w 8 && \
./examples/run.sh -m "stronghold" -l 32 -h 2048 -w 10 && \
./examples/run.sh -m "stronghold" -l 32 -h 2048 -w 12 && \
./examples/run.sh -m "stronghold" -l 32 -h 2048 -w 14
```

> **`./examples/case5_extract.sh ` and `./examples/case5_draw.sh ` help you analysis log files and print the relevant information only.**


