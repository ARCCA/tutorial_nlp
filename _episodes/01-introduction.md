---
title: "Introduction"
teaching: 10
exercises: 0
questions:
- "How do I access Hawk?"
- "How do I copy files?"
- "How do I run jobs?"
objectives:
- "Use a SSH terminal"
- "Use a file transfer program"
- "Use Slurm commands to query the system"
keypoints:
- "Understand the system and how it works"
---

> ## Training available!
>
> The training course [Supercomputing for Beginners]({{site.url}}/hpc-intro) and [Slurm Advanced
> Topics]({{site.url}}/slurm_advanced_topics)
> may be suitable to learn further.
{: .callout}

## What is an HPC system?

The words "cloud", "cluster", and "high-performance computing" are used a lot in different contexts
and with varying degrees of correctness. So what do they mean exactly? And more importantly, how do
we use them for our work?

The *cloud* is a generic term commonly used to refer to remote computing resources of any kind --
that is, any computers that you use but are not right in front of you. Cloud can refer to
machines serving websites, providing shared storage, providing webservices (such as e-mail or social
media platforms), as well as more traditional "compute" resources. An *HPC system* on the other hand,
is a term used to describe a network of computers. The computers in a cluster typically share a common
purpose, and are used to accomplish tasks that might otherwise be too big for any one computer.

> ## Registration
> 
> An account and membership of a project is required to run code on Supercomputing Wales resources. Please see:
> 
> [Getting Access](https://portal.supercomputing.wales/index.php/getting-access/)
{: .callout}

## Logging in

Go ahead and log in to the cluster: {{ site.host_name }} at {{ site.host_location }}.
```
{{ site.local_prompt }} ssh c.username@{{ site.host_login }}
```
{: .bash}

Remember to replace `c.username` with the username supplied by the instructors. You will be asked for
your password. But watch out, the characters you type are not displayed on the screen.

You are logging in using a program known as the secure shell or `ssh`. 
This establishes a temporary encrypted connection between your laptop and `{{ site.host_login }}`.
The word before the `@` symbol, e.g. `c.username` here, is the user account name that Lola has access 
permissions for on the cluster. 

## Where are we?

Very often, many users are tempted to think of a high-performance computing installation as one
giant, magical machine. Sometimes, people will assume that the computer they've logged onto is the
entire computing cluster. So what's really happening? What computer have we logged on to? The name
of the current computer we are logged onto can be checked with the `hostname` command. (You may also
notice that the current hostname is also part of our prompt!)

```
{{ site.host_prompt}} hostname
```
{: .bash}

```
{{ site.host_name }}
```
{: .output}

## Nodes

Individual computers that compose a cluster are typically called *nodes* (although you will also
hear people call them *servers*, *computers* and *machines*). On a cluster, there are different
types of nodes for different types of tasks. The node where you are right now is called the *head
node*, *login node* or *submit node*. A login node serves as an access point to the cluster. As a
gateway, it is well suited for uploading and downloading files, setting up software, and running
quick tests. **It should never be used for doing actual work**.

The real work on a cluster gets done by the *worker* (or *compute*) *nodes*. Worker nodes come in
many shapes and sizes, but generally are dedicated to long or hard tasks that require a lot of
computational resources.

All interaction with the worker nodes is handled by a specialized piece of software called a
scheduler (the scheduler used in this lesson is called {{ site.workshop_shed_name }}). We'll
learn more about how to use the scheduler to submit jobs next, but for now, it can also tell us
more information about the worker
nodes.

For example, we can view all of the worker nodes with the `{{ site.sched_info }}` command.

```
{{ site.host_prompt}} {{ site.sched_info }}
```
{: .bash}
```
{% include /snippets/12/info.snip %}
```
{: .output}

There are also specialized machines used for managing disk storage, user authentication, and other
infrastructure-related tasks. Although we do not typically logon to or interact with these machines
directly, they enable a number of key features like ensuring our user account and files are
available throughout the HPC system.

> ## Shared file systems
> This is an important point to remember: files saved on one node (computer) are often available
> everywhere on the cluster!
{: .callout}

## What's in a node? 

All of a HPC system's nodes have the same components as your own laptop or desktop: *CPUs* (sometimes
also called *processors* or *cores*), *memory* (or *RAM*), and *disk* space. CPUs are a computer's
tool for actually running programs and calculations. Information about a current task is stored in
the computer's memory. Disk refers to all storage that can be accessed like a file system. This is
generally storage that can hold data permanently, i.e. data is still there even if the computer has
been restarted.

{% include figure.html url="" max-width="40%" file="/fig/node_anatomy.png" alt="Node anatomy" caption="" %}

> ## Explore Your Computer
>
> Try to find out the number of CPUs and amount of memory available on your personal computer.
{: .challenge}

> ## Explore The Login Node
>
> Now we'll compare the size of your computer with the size of the login node: To see the number of
> processors, run:
>
> ```
> {{ site.host_prompt}} nproc --all
> ```
> {: .language-bash}
> 
> How about memory? Try running: 
>
> ```
> {{ site.host_prompt}} free -m
> ```
> {: .language-bash}
{: .challenge}



> ## Getting more information
>
> You can get more detailed information on both the processors and memory by using different commands.
>
> For more information on processors use `lscpu`
> ```
> {{ site.host_prompt}} lscpu
> ```
> {: .language-bash}
>
> For more information on memory you can look in the `/proc/meminfo` file:
> ```
> {{ site.host_prompt}} cat /proc/meminfo
> ```
> {: .language-bash}
{: .callout}

{% include /snippets/12/explore.snip %}

> ## Compare Your Computer, the Login Node and the Worker Node
>
> Compare your laptop's number of processors and memory with the numbers you see on the cluster 
> login node and worker node. Discuss the differences with your neighbor. What implications do
> you think the differences might have on running your research work on the different systems
> and nodes?
{: .challenge}

> ## Units and Language
> 
> A computer's memory and disk are measured in units called *Bytes* (one Byte is 8 bits). 
> As today's files and memory have grown to be large given historic standards, volumes are 
> noted using the [SI](https://en.wikipedia.org/wiki/International_System_of_Units) prefixes. 
> So 1000 Bytes is a Kilobyte (kB), 1000 Kilobytes is a Megabyte, 1000 Megabytes is a 
> Gigabyte etc. 
> 
> History and common language have however mixed this notation with a different meaning. 
> When people say "Kilobyte", they mean 1024 Bytes instead. In that spirit, a Megabyte are 
> 1024 Kilobytes. To address this ambiguity, the [International System of 
> Quantities](https://en.wikipedia.org/wiki/International_System_of_Quantities) 
> standardizes the binary prefixes (with base of 1024) by the prefixes kibi, mibi, gibi,
>  etc. For more details, see [here](https://en.wikipedia.org/wiki/Binary_prefix)
{: .callout}

> ## Differences Between Nodes
> Many HPC clusters have a variety of nodes optimized for particular workloads. Some nodes
> may have larger amount of memory, or specialized resources such as Graphical Processing Units
> (GPUs).
{: .callout}

## Job scheduler
An HPC system might have thousands of nodes and thousands of users. How do we decide who gets what
and when? How do we ensure that a task is run with the resources it needs? This job is handled by a
special piece of software called the scheduler. On an HPC system, the scheduler manages which jobs
run where and when.

The following illustration compares these tasks of a job scheduler to a waiter in a restaurant.
If you can relate to an instance where you had to wait for a while in a queue to get in to a 
popular restaurant, then you may now understand why sometimes your job do not start instantly
as in your laptop.

{% include figure.html max-width="75%" file="/fig/restaurant_queue_manager.svg"
alt="Compare a job scheduler to a waiter in a restaurant" caption="" %}


> ## Job scheduling roleplay (optional)
> 
> Your instructor will divide you into groups taking on different roles in the cluster (users,
> compute nodes and the scheduler). Follow their instructions as they lead you through this
> exercise. You will be emulating how a job scheduling system works on the cluster.
> 
> [*notes for the instructor here*](../guide)
{: .challenge}

The scheduler used in this lesson is {{ site.sched_name }}. Although {{ site.sched_name }} is not used everywhere,
running jobs is quite similar regardless of what software is being used. The exact syntax might change, but the
concepts remain the same.

## Running a batch job

The most basic use of the scheduler is to run a command non-interactively. Any command (or series 
of commands) that you want to run on the cluster is called a *job*, and the process of using a
scheduler to run the job is called *batch job submission*.

> ## Project codes
> 
> Project codes allow the resources used to be recorded to a project and
> provides information on what research is used which resources.  It allows
> for focussing attention on certain projects if issues are spotted.
>
> Can you find your available project codes on [mySCW](https://my.supercomputing.wales)?
{: .challenge}


In this case, the job we want to run is just a shell script. Let's create a demo shell script to 
run as a test.

> ## Creating our test job
> 
> Using your favorite text editor, create the following script and run it. Does it run on the
> cluster or just our login node?
>
>```
>#!/bin/bash --login
>
> echo 'This script is running on:'
> hostname
> sleep 120
> ```
> {: .bash}
{: .challenge}

If you completed the previous challenge successfully, you probably realise that there is a
distinction between running the job through the scheduler and just "running it". To submit this job
to the scheduler, we use the `{{ site.sched_submit }}` command.

```
[{{ site.host_prompt }} {{ site.sched_submit }} {{ site.sched_submit_options }} example-job.sh
```
{: .bash}
```
{% include /snippets/13/submit_output.snip %}
```
{: .output}

And that's all we need to do to submit a job. Our work is done -- now the scheduler takes over and
tries to run the job for us. While the job is waiting to run, it goes into a list of jobs called 
the *queue*. To check on our job's status, we check the queue using the command
`{{ site.sched_status }} {{ site.sched_flag_user }}`.

```
{{ site.host_prompt }} {{ site.sched_status }} {{ site.sched_flag_user }}
```
{: .bash}
```
{% include /snippets/13/statu_output.snip %}
```
{: .output}

We can see all the details of our job, most importantly that it is in the "R" or "RUNNING" state.
Sometimes our jobs might need to wait in a queue ("PENDING") or have an error. The best way to check
our job's status is with `{{ site.sched_status }}`. Of course, running `{{ site.sched_status }}` repeatedly to check on things can be
a little tiresome. To see a real-time view of our jobs, we can use the `watch` command. `watch`
reruns a given command at 2-second intervals. This is too frequent, and will likely upset your system
administrator. You can change the interval to a more reasonable value, for example 60 seconds, with the
`-n 60` parameter. Let's try using it to monitor another job.

```
{{ site.host_prompt }} {{ site.sched_submit }} {{ site.sched_submit_options }} example-job.sh
{{ site.host_prompt }} watch -n 60 {{ site.sched_status }} {{ site.sched_flag_user }}
```
{: .bash}

You should see an auto-updating display of your job's status. When it finishes, it will disappear
from the queue. Press `Ctrl-C` when you want to stop the `watch` command.

## Customising a job

The job we just ran used all of the scheduler's default options. In a real-world scenario, that's
probably not what we want. The default options represent a reasonable minimum. Chances are, we will
need more cores, more memory, more time, among other special considerations. To get access to these
resources we must customize our job script.

Comments in UNIX (denoted by `#`) are typically ignored. But there are exceptions. For instance the
special `#!` comment at the beginning of scripts specifies what program should be used to run it
(typically `/bin/bash`) possibly with switches such as `/bin/bash --login` to select a certain behaviour of the program. Schedulers like {{ site.workshop_sched_name }} also have a special comment
used to denote special scheduler-specific options. Though these comments differ from scheduler to
scheduler, {{ site.workshop_sched_name }}'s special comment is `{{ site.sched_comment }}`.
Anything following the `{{ site.sched_comment }}` comment is interpreted as an
instruction to the scheduler.

Let's illustrate this by example. By default, a job's name is the name of the script, but the
`{{ site.sched_flag_name }}` option can be used to change the name of a job.

Submit the following job (`{{ site.sched_submit }} {{ site.sched_submit_options }} example-job.sh`):

```
#!/bin/bash --login
{{ site.sched_comment }} {{ site.sched_flag_name }} new_name

echo 'This script is running on:'
hostname
sleep 120
```
{: .language-bash}

```
{{ site.host_prompt }} {{ site.sched_status }} {{ site.sched_flag_user }}
```
{: .bash}
```
{% include /snippets/13/statu_name_output.snip %}
```
{: .output}

Fantastic, we've successfully changed the name of our job!

> ## Setting up email notifications
> 
> Jobs on an HPC system might run for days or even weeks. We probably have better things to do than
> constantly check on the status of our job with `{{ site.sched_status }}`. Looking at the
> man page for `{{ site.sched_submit }}`, can you set up our test job to send you an email
> when it finishes?
> 
{: .challenge}

### Resource requests

But what about more important changes, such as the number of cores and memory for our jobs? One 
thing that is absolutely critical when working on an HPC system is specifying the resources 
required to run a job. This allows the scheduler to find the right time and place to schedule our 
job. If you do not specify requirements (such as the amount of time you need), you will likely be
stuck with your site's default resources, which is probably not what we want.

The following are several key resource requests:

{% include /snippets/13/stat_options.snip %}

Note that just *requesting* these resources does not make your job run faster! We'll talk more 
about how to make sure that you're using resources effectively in a later episode of this lesson.

> ## Submitting resource requests
>
> Submit a job that will use 1 full node and 5 minutes of walltime.
{: .challenge}

{% include /snippets/13/env_challenge.snip %}

Resource requests are typically binding. If you exceed them, your job will be killed. Let's use
walltime as an example. We will request 30 seconds of walltime, and attempt to run a job for two
minutes.

```
{% include /snippets/13/long_job.snip %}
```
{: .output}

Submit the job and wait for it to finish. Once it is has finished, check the log file.

```
{{ site.host_prompt }} {{ site.sched_submit }} {{ site.sched_submit_options }} example-job.sh
{{ site.host_prompt }} watch -n 60 {{ site.sched_status }} {{ site.sched_flag_user }}
{% include /snippets/13/long_job_cat.snip %}
```
{: .bash}
```
{% include /snippets/13/long_job_err.snip %}
```
{: .output}

Our job was killed for exceeding the amount of resources it requested. Although this appears harsh,
this is actually a feature. Strict adherence to resource requests allows the scheduler to find the
best possible place for your jobs. Even more importantly, it ensures that another user cannot use
more resources than they've been given. If another user messes up and accidentally attempts to use
all of the cores or memory on a node, {{ site.sched_name }} will either restrain their job
to the requested resources or kill the job outright. Other jobs on the node will be unaffected.
This means that one user cannot  mess up the experience of others, the only jobs affected by a
mistake in scheduling will be their
own.

## Cancelling a job

Sometimes we'll make a mistake and need to cancel a job. This can be done with the `{{ site.sched_del }}`
command. Let's submit a job and then cancel it using its job number (remember to change the 
walltime so that it runs long enough for you to cancel it before it is killed!).

```
{{ site.host_prompt }} {{ site.sched_submit }} {{ site.sched_submit_options }} example-job.sh
{{ site.host_prompt }} {{ site.sched_status }} {{ site.sched_flag_user }}
```
{: .bash}
```
{% include /snippets/13/del_job_output1.snip %}
```
{: .output}

Now cancel the job with it's job number. Absence of any job info indicates that the job has been
successfully cancelled.  Note that it might take a minute for the job to disappear from the queue.

```
{% include /snippets/13/del_job.snip %}
{{ site.host_prompt }} {{ site.sched_status }} {{ site.sched_flag_user }}
```
{: .bash}
```
{% include /snippets/13/del_job_output2.snip %}
```
{: .output}

{% include /snippets/13/del_multiple_challenge.snip %}

## Other types of jobs

Up to this point, we've focused on running jobs in batch mode. {{ site.sched_name }}
also provides the ability to start an interactive session.

There are very frequently tasks that need to be done interactively. Creating an entire job
script might be overkill, but the amount of resources required is too much for a login node to
handle. A good example of this might be building a genome index for alignment with a tool like
[HISAT2](https://ccb.jhu.edu/software/hisat2/index.shtml). Fortunately, we can run these types of
tasks as a one-off with `{{ site.sched_interactive }}`.

{% include /snippets/13/interactive_example.snip %}

{% include links.md %}

