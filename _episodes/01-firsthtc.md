---
title: "Run Your First HTC Jobs"
teaching: 10
exercises: 0
questions:
- "How can I use HTCondor to submit and track HTC work on the Open Science Grid?"
objectives:
- "Understand basic HTCondor job submission and job tracking in the queue."
keypoints:
- "HTCondor is software installed on our OSG Connect submit server that will run jobs on OSG execute servers."	 
- "The user-created submit file describes the computational work ('jobs') that HTCondor should run."	 
- "The `condor_q` command allows the user to track various batches and individual jobs."
---
### Background

The [Open Science Grid (OSG)](http://opensciencegrid.org/) is a collaboration of 
roughly 100 computing centers who share backfill computing capacity with one 
another for the purpose of executing large, high-throughput computing workloads. 
Like many HTC-oriented computing systems, the OSG uses the 
[HTCondor](https://research.cs.wisc.edu/htcondor/) compute scheduling 
software to allow users to submit and run computing work across this capacity. 

Because many campuses and organizations may not have their own submission point 
into the OSG, the [OSG Connect](https://osgconnect.net/) service provides access 
to a free submission point, as well as pre-installed software (like various versions of 
Matlab, Python, freesurfer, and others), for researchers at U.S. academic institutions, 
national labs, and non-profit organizations. Today, you'll be using your CogSci account 
on the OSG Connect training server to submit different types of jobs to the Open Science 
Grid using the HTCondor software.

Though we won't possibly be able to cover all of HTCondor's capabilities for users, 
there is a great [HTCondor User Tutorial](https://agenda.hep.wisc.edu/event/1201/session/4/contribution/5/material/slides/1.pdf) 
from the annual [HTCondor Week conference](https://agenda.hep.wisc.edu/event/1201/other-view) 
that you can sift through for even more of the most useful ways to submit 
and track jobs in an HTCondor pool.

Following today's lessons, you can also 
[request a full OSG Connect account](https://osgconnect.net/signup) if/when you think 
you might be ready to use OSG Connect for some of your own computing work. There are 
also numerous online guides for using OSG Connect with a full acount, available on the 
[OSG Connect Helpdesk](https://support.opensciencegrid.org/support/home).
 
### Set Up a Simple Set of 'dummy' HTCondor Jobs

So that we can focus on the basic steps of submitting a set of HTC jobs to HTCondor, 
we'll first create and submit a set of similar, VERY simple jobs to the HTCondor queue, 
and then track them in the queue.

First, let's create a directory just for this simple set of jobs:

~~~
$ mkdir FirstHTC
$ cd FirstHTC
~~~
{: .language-bash}

Now, let's create a basic shell script as the executable that each job should run. 
Use the `nano` text editor to create an empty file called `simple-job.sh`:

~~~
$ nano simple-job.sh
~~~
{: .language-bash}

Copy and paste the below contents into this file, then write out the changes 
and close the file:

~~~
#!/bin/bash
# 
# The below command will print the below message to a new output file
# based upon the first argument to this script, as well as the username 
# and server name that executed the script:
echo "This is job $1, which ran as `whoami` on `hostname`." > my$1.out
#
# The below commands will print the path and contents of the execution 
# directory to standard output:
pwd
ls
# END
~~~

Now, let's create a submit file to submit four jobs that will each run 
this executable, but with a different, numbered argument each time. Use 
`nano` again to create a new file named `simple-HTC.sub` with the below 
contents:

~~~
# A simple HTCondor submit file:
#
# Specify your executable script and arguments that each job should run.
#  The $(Process) variable is built-in to HTCondor as one way to distinguish 
#  different jobs, and will be a unique integer number for each job, 
#  starting with "0" and increasing for the number of jobs that we'll 'queue' 
# at the end of this file.
executable = simple-job.sh
arguments = $(Process)
#
# Let's also use $(Process) to specify unique filenames for files that HTCondor 
#  will store the standard error and standard output information to:
output = simple_$(Process).out
error = simple_$(Process).err
#
# HTCondor will also store information about the steps it's taking for each job
#  to a log file, if we tell it what that file should be named for each job:
log = simple_$(Process).log
#
# We'll estimate relatively small amounts of memory and disk space,
#  and one CPU core (the default), per job:
request_cpus = 1
request_memory = 1MB
request_disk = 1MB
#
# Tell HTCondor to run 4 instances of our job:
queue 4
~~~

Note that the above file would really only need to be about 10 lines long, if we 
didn't want all of those comments in it (though they're important for helping to 
explain this specific example).

### Before You Submit: View the HTCondor Queue and Available Slots

Before submitting our jobs to the HTCondor queue, let's first view the queue, 
which will also confirm for us that HTCondor is running on this server:

~~~
$ condor_q
~~~
{: .language-bash}

~~~


-- Schedd: training.osgconnect.net : <128.135.158.189:9618?... @ 07/23/18 21:51:00
OWNER BATCH_NAME      SUBMITTED   DONE   RUN    IDLE   HOLD  TOTAL JOB_IDS

0 jobs; 0 completed, 0 removed, 0 idle, 0 running, 0 held, 0 suspended
~~~

By default, HTCondor only shows us our own jobs, but we could see jobs from other 
users on **this** submit server with:

~~~
$ condor_q -all
~~~
{: .language-bash}

There probably aren't many jobs submitted yet. One more thing before we submit our 
jobs - let's look at the servers available in the "pool" of compute "slots" 
currently available to users of OSG Connect:

~~~
$ condor_status
~~~
{: .language-bash}

The top will look like this:
~~~
Name                              OpSys      Arch   State     Activity LoadAv Mem    ActvtyTime

slot1@amundsen.grid.uchicago.edu  LINUX      X86_64 Claimed   Busy      1.140 32768  0+00:42:54
slot2@amundsen.grid.uchicago.edu  LINUX      X86_64 Claimed   Busy      1.540 32768  0+00:42:54
bigmem-worker-1.grid.uchicago.edu LINUX      X86_64 Unclaimed Idle      0.000 64432  6+09:02:47
~~~

and the bottom will look something like:
~~~
slot1@htcondor-997969d8d-zlt87    LINUX      X86_64 Unclaimed Idle      0.060  6336  0+17:39:57
slot1_1@htcondor-997969d8d-zlt87  LINUX      X86_64 Claimed   Busy      0.000   512  0+01:34:24
slot1_2@htcondor-997969d8d-zlt87  LINUX      X86_64 Claimed   Busy      0.000   128  0+01:39:56
slot1_3@htcondor-997969d8d-zlt87  LINUX      X86_64 Claimed   Busy      0.000   512  0+02:47:04
slot1_4@htcondor-997969d8d-zlt87  LINUX      X86_64 Claimed   Busy      0.000   512  0+02:47:04
slot1@scott.grid.uchicago.edu     LINUX      X86_64 Claimed   Busy      1.150 32768  0+00:40:47
slot2@scott.grid.uchicago.edu     LINUX      X86_64 Claimed   Busy      0.860 32768  0+00:40:46

                     Machines Owner Claimed Unclaimed Matched Preempting  Drain

        X86_64/LINUX      205     0     164        41       0          0      0

               Total      205     0     164        41       0          0      0
~~~

Though it may not be obvious as you use OSG Connect, the names and 
total numbers of these slots will change over time, even increasing 
when there are more jobs in the queue. This is partially because 
OSG servers will come and go (depending on whether they're available for 
OSG-style backfill), and because an OSG submit point will purposefully 
go out and find more available servers when they're needed for waiting jobs.

Generally, you won't need to use `condor_status` unless you're trying to figure 
out why your jobs aren't running (perhaps there are no slots that are 'big' 
enough for the memory, disk, and cpu needs of your jobs). In that case, you 
could start by consulting the HTCondor User Tutorial (linked above under "Background") 
for more `condor_status` tips, or get in touch with the OSG Connect team.

### Submit Your Jobs to the Queue

Okay, let's submit our jobs to the HTCondor queue:

~~~
$ condor_submit simple-HTC.sub
~~~
{: .language-bash}
~~~
Submitting job(s)....
4 job(s) submitted to cluster 1191.
~~~

You'll see a different "cluster" number, which is a unique number assigned to 
**your** set of jobs that you just submitted (it does not refer to the idea of 
a compute "cluster" of servers that run jobs).

Right after submitting, you can view your jobs in the queue:
~~~
$ condor_q
~~~
{: .language-bash}
~~~


-- Schedd: training.osgconnect.net : <128.135.158.189:9618?... @ 07/23/18 22:15:12
OWNER    BATCH_NAME            SUBMITTED   DONE   RUN    IDLE  TOTAL JOB_IDS
cogsci50 CMD: simple-job.sh   7/23 22:15      _      _      4      4 1191.0-3

4 jobs; 0 completed, 0 removed, 4 idle, 0 running, 0 held, 0 suspended
~~~

If you catch your jobs in the queue before any of them have started running, 
you'll see all four of them represented in one line of the `condor_q` output. 
Once our jobs start running and completing, there will be counts showing up 
under "RUN" or "DONE".

To view a bit more information about each job on separate lines, we can ask 
`condor_q` not to batch our cluster of four jobs together:
~~~
$ condor_q -nobatch
~~~
{: .language-bash}
~~~


-- Schedd: training.osgconnect.net : <128.135.158.189:9618?... @ 07/23/18 22:22:03
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD
1191.0   cogsci50        7/23 22:22   0+00:00:00 I  0    0.0 simple-job.sh 0
1191.1   cogsci50        7/23 22:22   0+00:00:00 I  0    0.0 simple-job.sh 1
1191.2   cogsci50        7/23 22:22   0+00:00:00 I  0    0.0 simple-job.sh 2
1191.3   cogsci50        7/23 22:22   0+00:00:00 I  0    0.0 simple-job.sh 3

4 jobs; 0 completed, 0 removed, 4 idle, 0 running, 0 held, 0 suspended
~~~

Notice that we see more information about each job, including a unique job "ID" 
number, which corresponds to that job's 'cluster' and 'process' numbers, as 
determined when we submitted the job, and according to `queue 4` line in our 
submit file.

To view everyone's jobs and see them running (there should be a lot more total 
jobs in the queue):
~~~
$ condor_q -all
~~~
{: .language-bash}

If you keep running the above `condor_q` commands, you'll eventually see that 
a cluster of jobs will no longer show up in the `condor_q` output when 
all jobs of that cluster have completed.

In the next exercise, we'll look at the various files created by our jobs!

