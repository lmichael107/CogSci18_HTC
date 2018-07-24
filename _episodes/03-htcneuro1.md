---
title: "Setting Up a Neuroscience HTC Workload"
teaching: 10
exercises: 0
questions:
- "How can a real fMRI analysis be setup and launched?"
objectives:
- ""
keypoints:
- ""
---

## Applying HTC to solve a problem

So far, we have seen how to add jobs to HTCondor's queue, monitored their status using `condor_q`, and observed how each successful process returned a unique file.
How can we take these basic principles and apply them to a real problem?

### Elements of an experiment

While all experiments vary in their details, they all tend to share a few elements:

1. Functional imaging data for a set of participants.
2. Metadata that provides context for those images, including condition labels, filters, coordinates, and more.
3. An analysis procedure.
4. A set of instructions or parameters that define a particular analysis applied to particular data with reference to elements of metadata.

### Defining a workflow

In principle, each of the elements can take a variety of forms.
In order to define a workflow, however, one must define a set of formats and standards so that the elements can be combined and processing steps and interopertate.
In the example we will work through today, we are going to use a combination of elements that together comprise a WISC MVPA workflow.

### WISC MVPA

**W**hole-brain **I**maging with **S**parse **C**orrelations **M**ulti-**V**ariate **P**attern **A**nalysis defines the four elements as:

1. An item-by-voxel matrix for each subject, each stored as a named variable in a Matlab `.mat` file.
2. A structured array with a structure for each subject that contains information about condition labels, filters, and coordinates.
3. A Matlab program that expects data and metadata as in (1) and (2).
4. A `json` formatted structured-text file that encodes instructions that point (3) to specific elements of (1) and (2), and parameters the define exactly how the analysis should proceed.

WISC MVPA can also be used interactively (from within a Matlab script), but it is designed with HTC as the primary use case.
When run without arguments, `WISC_MVPA()` will look for a file named `params.json` in Matlab's current working directory.

WISC MVPA, as the name implies, will run various whole-brain MVPA analyses on preprocessed data.
This means that the data in the item-by-voxel `.mat` files will have been fully preprocessed before beginning the WISC MVPA workflow.

### Analysis on OSG

In order to run any analysis on OSG, the data and metadata must be hosted somewhere it can be accessed by HTCondor and transfered to the processes that need them.
There are a multitude of compatible ways to host ones data to accomplish this goal, and the way you do this will depend on your lab/university/institution's infrastructure.
The data we will be accessing today is hosted on a server maintained by the Center for High Throughput Computing at UW-Madison.

The analysis, of course, also requires that the code to be executed is accessible by HTCondor for the same reason.
Unlike the data, however, the code will typically be small enough that it can be hosted directly from the submit node (in your own home directory).
When working with Matlab code, there is an additional step.
Matlab code must be "compiled" using `mcc` before it can be used on a compute cluster.
This is simply because running compiled code against the Matlab Runtime Environment does not require a license instance, but running code interactively does.
Unfortunately, [OSG Connect does not have a license to use `mcc`](https://crcox.github.io/CogSci18_HTC/03-htcneuro1/index.html), the Matlab compiler.
However, your lab/univeristy/institution will very likely have a place to compile Matlab code (otherwise, you probably would not be developing in Matlab for your research!).

With your data hosted and code prepared and on the submit node, all that is left is to write instructions for the individual jobs.
When running hundreds or thousands of jobs, you do not want to do this by hand.
There are many possible solutions to this problem.
We are going to use one that I developed and have been using for years: the `setupJobs` command in my `InputSetup` repository.
It will translate a single YAML formatted text file (chosen because it allows comments and strives for human-readability) into as many json files as necessary, each within a numbered directory.
The results from each job will be returned into the directory corresponding to its parameter file.

But before diving into the functionality of `setupJobs`, let's setup and run an analysis!

## Experiment overview

For this example, we are going to preform the hyperparameter search step for sparse whole-brain classifiers learning to discriminate face from non-face trials in 10 subject.
The data are a portion of what was published by [Lewis-Peacock and Postle, 2008](https://www.ncbi.nlm.nih.gov/pubmed/18753378).

## Setting up a set of jobs with `setupJobs`

First, create a small directory tree for our demo and download some files:

~~~
$ mkdir -p ~/WISC_MVPA/lasso/performance/tune
$ cd ~/WISC_MVPA/lasso/performance/tune
$ wget http://proxy.chtc.wisc.edu/SQUID/crcox/CogSci2018_HTC/stub.yaml
$ wget http://proxy.chtc.wisc.edu/SQUID/crcox/CogSci2018_HTC/stub_hb.yaml
$ wget http://proxy.chtc.wisc.edu/SQUID/crcox/CogSci2018_HTC/WISC_MVPA
$ wget http://proxy.chtc.wisc.edu/SQUID/crcox/CogSci2018_HTC/run_WISC_MVPA_OSG.sh
~~~
{: .language-bash}

The `stub.yaml` file contains everything necessary to define, in this case, 400 jobs.
We will circle back to what it actually contains.
Note that we also pulled down a `stub_hb.yaml` file.
In practice, a fully functional Python environment for `setupJobs` will have `numpy` and `pandas`.
These can be installed in your home directory in a virtual environment on OSG Connect, but they take a few minutes to install so we are not going to bother with that now.
The `stub_hb.yaml` is an intermediate file produced after a first round of processing `stub.yaml`, where some extra parameters are filled in.
Important for our demo is that `stub_hb.yaml` can be parsed without `numpy` or `pandas`.

Now, we will pass `stub_hb.yaml` to `setupJobs`, with a flag that will ensure that file paths are adapted for the execute node.

~~~
$ setupJobs -l stub.yaml
~~~
{: .language-bash}

~~~
Composing 400 individual jobs...
Done.
~~~
{: .output}

~~~
$ ls
~~~
{: .language-bash}

~~~
000  024  048  072  096  120  144  168  192  216  240  264  288  312  336  360  384
001  025  049  073  097  121  145  169  193  217  241  265  289  313  337  361  385
002  026  050  074  098  122  146  170  194  218  242  266  290  314  338  362  386
003  027  051  075  099  123  147  171  195  219  243  267  291  315  339  363  387
004  028  052  076  100  124  148  172  196  220  244  268  292  316  340  364  388
005  029  053  077  101  125  149  173  197  221  245  269  293  317  341  365  389
006  030  054  078  102  126  150  174  198  222  246  270  294  318  342  366  390
007  031  055  079  103  127  151  175  199  223  247  271  295  319  343  367  391
008  032  056  080  104  128  152  176  200  224  248  272  296  320  344  368  392
009  033  057  081  105  129  153  177  201  225  249  273  297  321  345  369  393
010  034  058  082  106  130  154  178  202  226  250  274  298  322  346  370  394
011  035  059  083  107  131  155  179  203  227  251  275  299  323  347  371  395
012  036  060  084  108  132  156  180  204  228  252  276  300  324  348  372  396
013  037  061  085  109  133  157  181  205  229  253  277  301  325  349  373  397
014  038  062  086  110  134  158  182  206  230  254  278  302  326  350  374  398
015  039  063  087  111  135  159  183  207  231  255  279  303  327  351  375  399
016  040  064  088  112  136  160  184  208  232  256  280  304  328  352  376  queue_input.csv
017  041  065  089  113  137  161  185  209  233  257  281  305  329  353  377  queue_input.header
018  042  066  090  114  138  162  186  210  234  258  282  306  330  354  378  shared
019  043  067  091  115  139  163  187  211  235  259  283  307  331  355  379  stub.yaml
020  044  068  092  116  140  164  188  212  236  260  284  308  332  356  380
021  045  069  093  117  141  165  189  213  237  261  285  309  333  357  381
022  046  070  094  118  142  166  190  214  238  262  286  310  334  358  382
023  047  071  095  119  143  167  191  215  239  263  287  311  335  359  383
~~~
{: .output}

~~~
$ ls ???/ shared/ | head
~~~
{: .language-bash}

~~~
000/:
params.json

001/:
params.json

002/:
params.json

...

shared/:
run_WISC_MVPA_OSG.sh  WISC_MVPA
~~~
{: .output}


### Review of the output

The consequence of running `setupJobs` is 400 zero-padded numbered directories with zero-based indexing (000--399), each containing a `params.json` file.
A `shared/` directory was also produced, which contains the compiled Matlab code `WISC_MVPA` and a bash script for executing it where HTCondor sends it, `run_WISC_MVPA_OSG.sh`.
Finally, `queue_input.csv` and `queue_input.header` were written.
These files are very important, but also very simple.

### What is `queue_input.csv`?

This file contains a table, where each row corresponds to one of the 400 numbered directories, and each column (except the first) correponds to a file that needs to be transfered along with the job but does not exist in a numbered directory or the shared folder.
Notice that these are full URLs.
Each points to a publicly accessible location on the internet.
If you were to copy one of those URLs into your web browser, it would begin downloading the file to your computer.
HTCondor will do the same with these URLs once it picks a location to deploy a particular job.

Everything that the job needs to run must be sent there, and HTCondor will send only what you tell it to send.
This will become more clear once we begin updating our Submit File to pull all these pieces together.

### What is `run_WISC_MVPA_OSG.sh`?

This script will setup the Matlab environment on the execute node and launch the actual analysis.

~~~
#!/bin/bash
# run_WISC_MVPA_OSG.sh

set -e
set -x

EXECUTABLE=$1
# Run the Matlab application
# NB: This script needs to run to the end, even if there are errors.
set +e
source /cvmfs/oasis.opensciencegrid.org/osg/modules/lmod/current/init/bash
set -e

module load matlab/2015b

chmod +x ${EXECUTABLE}
eval "./${EXECUTABLE}"
~~~
{: .language-bash}

### Composing the Submit File

For sake of continuity, we are going to build off the "hello world" submit file use in the prior example.
This file, `simple-HTC.sub`, is reproduced below for reference.

~~~
# A simple HTCondor submit file:
#
executable = simple-job.sh
arguments = $(Process)
#
output = simple_$(Process).out
error = simple_$(Process).err
log = simple_$(Process).log
#
# We'll estimate relatively small amounts of memory and disk space,
#  and one CPU core (the default), per job:
request_cpus = 1
request_memory = 1MB
request_disk = 1MB
#
queue 4
~~~

~~~
# A HTCondor submit file for running WISC MVPA:
#
executable = ./shared/run_WISC_MVPA_OSG.sh
arguments = "WISC_MVPA"
#
initialdir = $(jobdir)
output = wisc_mvpa.out
error = wisc_mvpa.err
log = wisc_mvpa.log
#
request_cpus = 1
request_memory = 2MB
request_disk = 5MB
#
requirements = (OSGVO_OS_STRING == "RHEL 6" || OSGVO_OS_STRING == "RHEL 7") && Arch == "X86_64" && HAS_MODULES == True
#
should_transfer_files = YES
transfer_input_files = ./shared,$(jobdir),$(data),$(meta)
#
queue jobdir data meta from queue_input.csv

~~~

1. From the perspective of HTCondor, `run_WISC_MVPA_OSG.sh` is the thing that
 needs to be executed. Even though you may think of our analysis code as "the
 executable", remember that a bit of setup needs to be done on the execute node
 before Matlab will run properly. So HTCondor will execute `run_WISC_MVPA_OSG.sh`
 as soon as it is ready, and that will in turn run `WISC_MVPA` (the name of
 which is passed as an argument. This is an argument to `run_WISC_MVPA_OSG.sh`,
 and so that bash script needs to be written so that it accepts the argument).

2. This section is almost thesame as above, except that rather than naming each
 output file based on the process that generated it, we want files to be
 returned into the directory associated with that process. **initialdir** defines
 where a particular job is being run from, and all files will be returned to
 that location. Notice that we that the value assigned to initialdir is a
 variable, `$(jobdir)`. This works like `$(Process)` did in the simple submit file
 above, except that rather than being a variable that is automatically generated
 by HTCondor, `$(jobdir)` is a variable defined within this submit file (on the
 *queue* line below).

3. **requirements** will ensure that jobs are only sent to machines that meet the
   listed conditions. These conditions state that the machine should be running
   Red Hat Linux 6 or 7, have a 64bit CPU, and have the OSG Modules installed.

4. **should_transfer_files** indicates (as the name implies) whether files should
   be sent between submit and execute nodes.

5. **transfer_input_files** This is a list of files and directories to be
   transfered to the execute node. Paths to files can either point to locations on
   the file system (using relative or absolute paths) or be an HTTP address.
   Notice here, too, that this argument is composed of several variables. These
   are defined on the **queue** line below.

   Because the ./shared directory is listed here, the releveant shared code
   will be sent along.

   Because the `$(jobdir)` variable is listed here, the params.json file will
   containing job instructions will be sent along.

   Because the `$(data)` and `$(meta)` variables are listed, the data and metadata
   will be downloaded from the specified HTTP addresses.

6.  The queue command is the same as before, but rather than defining a number
 which specifies how many jobs to add to the queue, it will look in the
 `queue_input.csv` file for the information. Each row in `queue_input.csv` will
 correspond to an entry in the queue, and the comma-separated elements of that
 row will be parsed into separate variables. The names of those variables are
 listed immediately after the queue command, in order (jobdir corresponds to
 column 1, data corresponds to column 2, etc). These variables can be used
 throughout the submit file. In effect, each row in `queue_input.csv` will
 correspond to a different version of the submit file, with the values from that
 row filled into the variable references throughout the file. These 400 submit
 files are never actuall created, however. HTCondor simply knows what to do, and
 will add 400 unique jobs to the queue.

With only small tweaks, this submit file should handle most needs.
The full documentation of submit file variables can be found [here](http://research.cs.wisc.edu/htcondor/manual/v8.7/labelmancondorsubmitCondorsubmit.html#x149-109000012), and some variations of the submit file are demonstrated [here](http://www.chtc.cs.wisc.edu/submit_variations.shtml).

## Launching the jobs

Just as before, once the submit file is ready, the jobs can be launched by running.

~~~
$ condor_submit wisc_mvpa.sub
~~~
{: .language-bash}

As noted previously, we can get snapshots of the state of our jobs using `condor_q`.
If you want a more interactive view, try:

~~~
$ connect watch
~~~
{: .language-bash}

This is a command available on OSG connect that will effectively run `condor_q` every 2 seconds.
Aside from being a hypnotic procrastination aid, it can help to monitor the initial stages of a large batch: are jobs leaving the "running" column faster than expected and not getting tallied in the "done" column?
There is a problem!
Use `condor_rm {osgconnect username}` to remove all jobs.
Time to seek out the bug.

While those jobs run, we will return to `submitJobs` and `stub.yaml` to get a better idea of how that works.

