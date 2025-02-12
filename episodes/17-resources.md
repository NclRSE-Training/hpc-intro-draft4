---
title: "Using resources effectively"
teaching: 10
exercises: 30
---


::: questions
 - How do we monitor our jobs?
 - How can I get my jobs scheduled more easily?
:::

::: objectives
 - Understand how to look up job statistics and profile code.
 - Understand job size implications.
:::

We've touched on all the skills you need to interact with an HPC cluster:
logging in over SSH, loading software modules, submitting parallel jobs, and
finding the output. Let's learn about estimating resource usage and why it
might matter. To do this we need to understand the basics of *benchmarking*.
Benchmarking is essentially performing simple experiments to help understand
how the performance of our work varies as we change the properties of the
jobs on the cluster - including input parameters, job options and resources used.

::: discussion
## Our example
In the rest of this episode, we will use an example parallel application that calculates 
an estimate of the value of Pi. Although this is a toy problem, it exhibits all the properties of a full
parallel application that we are interested in for this course.

The main resource we will consider here is the use of compute core time as this is the
resource you are usually charged for on HPC resources. However, other resources - such
as memory use - may also have a bearing on how you choose resources and constrain your
choice.

For those that have come across HPC benchmarking before, you may be aware that people
often make a distinction between *strong scaling* and *weak scaling*:

- Strong scaling is where the problem size (i.e. the *application*) stays the same 
  size and we try to use more cores to solve the problem faster.
- Weak scaling is where the problem size increases at the same rate as we increase
  the core count so we are using more cores to solve a larger problem.

Both of these approaches are equally valid uses of HPC. This example looks at strong scaling. 

:::

Before we work on benchmarking, it is useful to define some terms for the example we will
be using

  - **Program** The computer program we are executing (`pi-mpi.py` in the examples below)
  - **Application** The combination of computer program with particular input parameters

## Accessing the software and input

```{r, child=paste(snippets, '/resources/pi-mpi-details.Rmd', sep=''), eval=TRUE}
```

## Baseline: running in serial

Before starting to benchmark an application to understand what resources are best to use,
you need a *baseline* performance result. In more formal benchmarking, your baseline
is usually the minimum number of cores or nodes you can run on. However, for understanding
how best to use resources, as we are doing here, your baseline could be the performance on
any number of cores or nodes that you can measure the change in performance from.

Our `pi-mpi.py` application is small enough that we can run a serial (i.e. using a single core)
job for our baseline performance so that is where we will start

::: challenge
## Run a single core job
Write a job submission script that runs the `pi-mpi.py` application on a single core. You
will need to take an initial guess as to the walltime to request to give the job time 
to complete. Submit the job and check the contents of the STDOUT file to see if the 
application worked or not.

::: solution

```{r, child=paste(snippets, '/resources/serial-submit.Rmd', sep=''), eval=TRUE}
```

Output in the job log should look something like:

```output
Generating 10000000 samples.
Rank 0 generating 10000000 samples on host nid001246.
Numpy Pi:  3.141592653589793
My Estimate of Pi:  3.1416708
1 core(s), 10000000 samples, 228.881836 MiB memory, 0.423903 seconds, -0.002487% error
```
:::
:::

Once your job has run, you should look in the output to identify the performance. Most 
HPC programs should print out timing or performance information (usually somewhere near
the bottom of the summary output) and `pi-mpi.py` is no exception. You should see two 
lines in the output that look something like:

```bash
256 core(s), 100000000 samples, 2288.818359 MiB memory, 0.135041 seconds, -0.004774% error
Total run time=0.18654435999997077s
```

You can also get an estimate of the overall run time from the final job statistics. If
we look at how long the finished job ran for, this will provide a quick way to see
roughly what the runtime was. This can be useful if you want to know quickly if a 
job was faster or not than a previous job (as you do not have to find the output file
to look up the performance) but the number is not as accurate as the performance recorded
by the application itself and also includes static overheads from running the job
(such as loading modules and startup time) that can skew the timings. To do this on
use ``r config$sched$hist` `r config$sched$flag$histdetail`` with the job ID, e.g.:

```bash
`r config$remote$prompt_work` `r config$sched$hist` `r config$sched$flag$histdetail` 12345
```
```output
```{r, child=paste(snippets, '/resources/job-detail.Rmd', sep=''), eval=TRUE}
```
```

## Running in parallel and benchmarking performance

We have now managed to run the `pi-mpi.py` application using a single core and have a baseline
performance we can use to judge how well we are using resources on the system.

Note that we also now have a good estimate of how long the application takes to run so we can
provide a better setting for the walltime for future jobs we submit. Lets now look at how
the runtime varies with core count.

```{r, child=paste(snippets, '/resources/runtime-exercise.Rmd', sep=''), eval=TRUE}
```

## Understanding the performance

Now we have some data showing the performance of our application we need to try and draw some
useful conclusions as to what the most efficient set of resources are to use for our jobs. To
do this we introduce two metrics:

  - **Actual speedup** The ratio of the baseline runtime (or runtime on the lowest core count)
    to the runtime at the specified core count. i.e. baseline runtime divided by runtime
    at the specified core count.
  - **Ideal speedup** The expected speedup if the application showed perfect scaling. i.e. if
    you double the number of cores, the application should run twice as fast.
  - **Parallel efficiency** The fraction of *ideal speedup* actually obtained for a given
    core count. This gives an indication of how well you are exploiting the additional resources
    you are using.

We will now use our performance results to compute these two metrics for the sharpen application
and use the metrics to evaluate the performance and make some decisions about the most 
effective use of resources.

```{r, child=paste(snippets, '/resources/perf-exercise.Rmd', sep=''), eval=TRUE}
```

## Tips

Here are a few tips to help you use resources effectively and efficiently on HPC systems:

 - Know what your priority is: do you want the results as fast as possible or are you happy
  to wait longer but get more research for the resources you have been allocated?
 - Use your real research application to benchmark but try to shorten the run so you can turn
  around your benchmarking runs in a short timescale. Ideally, it should run for 10-30 minutes;
  short enough to run quickly but long enough so the performance is not dominated by static startup
  overheads (though this is application dependent). Ways to do this potentially include, for example:
  using a smaller number of time steps, restricting the number of SCF cycles, restricting the
  number of optimisation steps.
 - Use basic benchmarking to help define the best resource use for your application. One 
  useful strategy: take the core count you are using as the baseline, halve the number of
  cores/nodes and rerun and then double the number of cores/nodes from your baseline and
  rerun. Use the three data points to assess your efficiency and the impact of different
  core/node counts.

::: keypoints
 - "The smaller your job, the faster it will schedule."
:::
