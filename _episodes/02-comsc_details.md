---
title: "Resources for School of Computer Science"
teaching: 10
exercises: 0
questions:
- "What is available for me?"
- "How do I select resources?"
- "How do I install software?"
objectives:
- "Understand the resources available as a member of School of Computer Science"
- "Use the resources effectively"
keypoints:
- "School of Computer Science has invested in resources, it is there to be used!"
---

## Dedicated resources

Anyone with funding can purchase hardware to connect into Hawk and use its existing filesystems, job scheduler and
ARCCA provides the management of the hardware.  A number of groups have purchased dedicated resource and School of Computer Science is
no different.  There are currently 2 nodes that can be used:

```
$ sinfo -p c_gpu_comsc1,c_gpu_diri1
```
{: .language-bash}

Will provide information on the partitions

```
$ sinfo -p c_gpu_comsc1,c_gpu_diri1
PARTITION    AVAIL  TIMELIMIT  NODES  STATE NODELIST
c_gpu_comsc1    up 3-00:00:00      1    mix ccs9201
c_gpu_diri1     up 3-00:00:00      1   idle ccs9202
```
{: .output}

To restrict access to the partitions we place access control on them usually by project code.  The current projects with
access can be seen with:

```
$ scontrol show partition c_gpu_comsc1
```
{: .language-bash}

And will produce:

```
PartitionName=c_gpu_comsc1
   AllowGroups=scw1001,scw1140,scw1390,scw1395,scw1077,scw1442,scw1458,scw1367,scw1377,scw1496,scw1572,scw1592 AllowAccounts=scw1001,scw1140,scw1390,scw1395,scw1077,scw1442,scw1458,scw1367,scw1377,scw1496,scw1572,scw1592 AllowQos=ALL
   AllocNodes=ALL Default=NO QoS=normal
   DefaultTime=NONE DisableRootJobs=YES ExclusiveUser=NO GraceTime=0 Hidden=NO
   MaxNodes=UNLIMITED MaxTime=3-00:00:00 MinNodes=0 LLN=NO MaxCPUsPerNode=UNLIMITED
   Nodes=ccs9201
   PriorityJobFactor=1 PriorityTier=1 RootOnly=NO ReqResv=NO OverSubscribe=NO
   OverTimeLimit=NONE PreemptMode=OFF
   State=UP TotalCPUs=40 TotalNodes=1 SelectTypeParameters=NONE
   JobDefaults=(null)
   DefMemPerCPU=9550 MaxMemPerNode=UNLIMITED
   TRESBillingWeights=CPU=0,Mem=0G
```
{: .output}

To get access we require permission from the owner of the partition.  Contact ARCCA and we can get things set-up for
you.

## Running a job on dedicated resource

Just like the previous section we can run using with project code `scwXXXX`:

```
$ sbatch -c 1 -p c_gpu_comsc1 -A scwXXXX
```
{: .language-bash}

This will reserve 1 CPUs/cores for your job.  What about the GPU?  We need to tell the scheduler to reserve the GPU.

```
$ sbatch -c 1 -p c_gpu_comsc1 -A scwXXXX --gres=gpu:1
```
{: .language-bash}

This will now reserve 1 GPU card for your job.  Another job can use the other GPU if required.

These can be included in a script:

```
#!/bin/bash --login
#SBATCH -c 1
#SBATCH -p c_gpu_comsc1
#SBATCH -A scwXXXX
#SBATCH --gres=gpu:1

```
{: .language-bash}

## Software

Software can be loaded through the `module` system.  The modules allow software to be loaded into your environment.
Modules are requested to be installed where ARCCA can install them for users to share.

One common module is:

```
$ module load anaconda/2020.02
```
{: .language-bash}

This is a recent Anaconda install (although should be updated again soon).  Usually conda will create environments in
the users home directory.  This can cause problems so we recommend regularly cleaning and running `conda clean -a`.

Another useful feature is the Singularity container system.  Singularity is similar to docker and can run software built
for other operating systems.  For example, software built and installed in Ubuntu can be placed inside a Singularity
container and run on a Redhat system.  Singularity is designed for HPC use to keep its features basic - there is no real
virtualisation taking place, just containerising the processes and filesystem.

> ## Training available!
>
> The training course [An Introduction to Singularity and Containers]({{site.url}}/intro_singularity)
> may be suitable to learn further.
{: .callout}

The following will provide the Singularity command

```
$ module load singularity
```
{: .language-bash}

The following will pull the latest Python docker container. 

```
$ singularity pull docker://python
```
{: .language-bash}

And the following will run Python in the container

```
$ singularity exec python_latest.sif python --version
```
{: .language-bash}

This will print:

```
Python 3.9.1
```
{: .output}

## Available GPU devices
There are currently two types of GPU devices available to our uses on two partitions, *gpu* and gpu_v100 which offer *P100* and *V100* generation NVIDIA cards respectively.  Partitions with `gpu` in the name will be the same as the `gpu` partition, `v100` signifies the newer models.

<table>
 <thead>
  <tr>
   <th>Partition name</th>
   <th>Number of nodes</th>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td>gpu</td>
   <td>13</td>
  </tr>
  <tr>
   <td>gpu_v100</td>
   <td>15</td>
  </tr>
 </tbody>
  <tfoot style="text-align:right">
   <tr>
     <td colspan="2"> All GPU nodes have 2 GPU devices </td>
   </tr>
  </tfoot>
</table>

The main difference between these devices is on the availability of Tensor cores on the *V100*. Tensor cores are a new type of programmable core exclusive to GPUs based on the Volta architecture that run alongside standard CUDA cores. Tensor cores can accelerate mixed-precision matrix multiply and accumulate calculations in a single operation. This capability is specially significant for AI/DL/ML applications that rely on large matrix operations.
<table>
 <tr>
  <th>Characteristic</th>
  <th>Volta</th>
  <th>Pascal</th>
 </tr>
 <tr>
  <td>Tensor cores</td>
  <td>640</td>
  <td>0</td>
 </tr>
 <tr>
  <td>Cuda cores</td>
  <td>5120</td>
  <td>3584</td>
 </tr>
 <tr>
  <td>Memory (Gb)</td>
  <td>16</td>
  <td>16</td>
 </tr>
</table>

## Requesting GPUs
Slurm controls access to the GPUs on a node such that access is only granted when the resource is requested specifically (i.e. is not implicit with processor/node count), so that in principle it would be possible to request a GPU node without GPU devices but this would bad practice.  Slurm models GPUs as a Generic Resource (GRES), which is requested at job submission time via the following additional directive:

~~~
#SBATCH --gres=gpu:2
~~~
{: .language-bash}

This directive instructs Slurm to allocate two GPUs per allocated node, to not use nodes without GPUs and to grant access.

On your job script you should also point to the desired GPU enabled partition:
~~~ 
#SBATCH -p gpu  # to request P100 GPUs
# Or
#SBATCH -p gpu_v100 # to request V100 GPUs
~~~
{: .language-bash}

It is then possible to use CUDA enabled applications or the CUDA toolkit modules themselves, modular environment examples being:

~~~
module load CUDA/9.1
module load gromacs/2018.2-single-gpu
~~~
{: .language-bash}

### GPU compute modes

NVIDIA GPU cards can be operated in a number of Compute Modes. In short the difference is whether multiple processes (and, theoretically, users) can access (share) a GPU or if a GPU is exclusively bound to a single process. It is typically application-specific whether one or the other mode is needed, so please pay particular attention to example job scripts. GPUs on SCW systems default to ‘shared’ mode.

Users are able to set the Compute Mode of GPUs allocated to their job through a pair of helper scripts that should be called in a job script in the following manner:

To set exclusive mode:
~~~
clush -w $SLURM_NODELIST "sudo /apps/slurm/gpuset_3_exclusive"
~~~
{: .language-bash}
And to set shared mode (although this is the default at the start of any job):
~~~
clush -w $SLURM_NODELIST "sudo /apps/slurm/gpuset_0_shared"
~~~
{: .language-bash}
To query the Compute Mode:
~~~
clush -w $SLURM_NODELIST "nvidia-smi -q|grep Compute"
~~~
{: .language-bash}

## Compiling with CUDA libraries

CUDA libraries are not accessible from general compute nodes (*compute*, *htc*, *highmem*) but they are on *dev* and login nodes for the purpose of testing and compilation. Trying to load CUDA libraries on these partitions would result in the following error:

```
ERROR: CUDA is not available. GPU detected is unknown
```
{: .error}

What this means is that you are capable of building your CUDA application on the login nodes and test basic functionality on *dev*, but to test actual GPU work you will need to submit your job to *gpu* or *gpu_v100*.

Current CUDA versions available on our systems are:

```
$ module avail CUDA
--------------------------- /apps/modules/libraries ----------------------------
CUDA/10.0 CUDA/10.2 CUDA/11.2 CUDA/9.0  CUDA/9.2
CUDA/10.1 CUDA/11   CUDA/8.0  CUDA/9.1
```
{: .output}

The latest NVIDIA driver version on the GPU nodes is 460.27.04 for CUDA 11.2  which backwards compatible with prior versions of CUDA. 

## Running a GPU job
Here is a job script to help you submit a "hello-world" GPU job:

~~~
#!/bin/bash --login
#SBATCH --job-name=gpu.example
#SBATCH --error=%x.e.%J
#SBATCH --output=%x.o.%J
#SBATCH --partition=gpu_v100
#SBATCH --time=00:10:00
#SBATCH --ntasks=40
#SBATCH --ntasks-per-node=40
#SBATCH --gres=gpu:2
#SBATCH --account=scw1148

clush -w $SLURM_NODELIST "sudo /apps/slurm/gpuset_3_exclusive"

module purge
module load keras/2.3.1
module list

test=imdb
input_dir=$SLURM_SUBMIT_DIR
WDPATH=/scratch/$USER/gpu.example.${test}.$SLURM_JOBID
rm -rf ${WDPATH}
mkdir -p ${WDPATH}
cd ${WDPATH}
cp ${input_dir}/${test}.py ${WDPATH}

start="$(date +%s)"
time python -u -i ${test}.py
stop="$(date +%s)"
finish=$(( $stop-$start ))
echo gpu test ${test} $SLURM_JOBID Job-Time $finish seconds
echo Keras End Time is `date`
~~~
{: .language-bash}

The above example is taken from "Deep Learning with Python" by Francois Chollet and explores a Deep Learning Two-class classification problem. In specific, the purpose of the model is to classify movie review as positive or negative based on the context of the reviews.

> ## Running a GPU job...
>
> Try running the above example.
> <pre style="color: silver; background: black;">
> $ sbatch gpu.example.q
> </pre>
> Does it work? Try submitting it to a non-GPU node. Is there a big difference in timing? Take into account that this is a very model. However, in real research problems, using GPU enabled applications have the potential to reduce significantly computational time.
{: .challenge}

Now we have covered the introduction and resources available, lets move to an example.




{% include links.md %}

