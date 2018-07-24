---
title: "Reviewing HTCondor Job Files"
teaching: 10
exercises: 0
questions:
- "How do I know that my jobs completed successfully?"
- "What files were created by my jobs and what information do they contain?"
objectives:
- "Examine job-created files."
- "Consult the log file for first indications of problems and for future job resource requests."
- "Understand the contents of a job's execution directory."
keypoints:
- "Forthcoming"
---

### What Job Files Were Created?

Our jobs have finished running, so they're no longer visible 
in the `condor_q` output, but let's see what files were created. 
Make sure you're still in the `FirstHTC` directory within which 
you submitted the jobs with `condor_submit`, then examine your 
directory contents:

~~~
$ ls
~~~
{: .language-bash}
~~~
my0.out  my3.out       simple_0.out  simple_1.out  simple_2.out  simple_3.out
my1.out  simple_0.err  simple_1.err  simple_2.err  simple_3.err  simple-HTC.sub
my2.out  simple_0.log  simple_1.log  simple_2.log  simple_3.log  simple.sh
~~~

That's quite a few files, so let's look at a few of them at a time. 
Taking a look at our submit file and executable script will help us remember 
what's what:

~~~
$ cat *.sub
~~~
{: .language-bash}
~~~
$ cat *.sh
~~~
{: .language-bash}

We're basically looking for any instance of a filename that 
is listed in either file.

**So the files we should see in our directory include:**

Submit and executable files, which we created before-hand:
- `simple.sh`, our executable
- `simple-HTC.sub`, our submit file queueing four jobs using the executable

Some "metadata" files created for each of our jobs (specified in the submit file with `output`, `error`, and `log`, and numbered using each job's 'Process' value):
- `simple_*.out`, where the terminal or "standard" output from the executable is saved
- `simple_*.err`, where (standard) error information from the executable is saved
- `simple_*.log`, where HTCondor has recorded the steps of running the job for us

And a single output file produced by our executable, for each job:
- `my*.out`, which was created by our executable

Before looking at each of the job-specific files for one job, 
let's check to see that the size of each file type is consistent 
across jobs. Doing so will give us an immediate idea of whether 
any one of the jobs seemed to produce different results:

~~~
$ ls -lh
~~~
{: .language-bash}
~~~
-rw-r--r-- 1 cogsci50 cogsci50   55 Jul 24 14:15 my0.out
-rw-r--r-- 1 cogsci50 cogsci50   55 Jul 24 14:15 my1.out
-rw-r--r-- 1 cogsci50 cogsci50   55 Jul 24 14:15 my2.out
-rw-r--r-- 1 cogsci50 cogsci50   59 Jul 24 14:15 my3.out
-rw-r--r-- 1 cogsci50 cogsci50    0 Jul 24 14:15 simple_0.err
-rw-r--r-- 1 cogsci50 cogsci50 1.2K Jul 24 14:15 simple_0.log
-rw-r--r-- 1 cogsci50 cogsci50   54 Jul 24 14:15 simple_0.out
-rw-r--r-- 1 cogsci50 cogsci50    0 Jul 24 14:15 simple_1.err
-rw-r--r-- 1 cogsci50 cogsci50 1.2K Jul 24 14:15 simple_1.log
-rw-r--r-- 1 cogsci50 cogsci50   54 Jul 24 14:15 simple_1.out
-rw-r--r-- 1 cogsci50 cogsci50    0 Jul 24 14:15 simple_2.err
-rw-r--r-- 1 cogsci50 cogsci50 1.2K Jul 24 14:15 simple_2.log
-rw-r--r-- 1 cogsci50 cogsci50   54 Jul 24 14:15 simple_2.out
-rw-r--r-- 1 cogsci50 cogsci50    0 Jul 24 14:15 simple_3.err
-rw-r--r-- 1 cogsci50 cogsci50 1.2K Jul 24 14:15 simple_3.log
-rw-r--r-- 1 cogsci50 cogsci50   54 Jul 24 14:15 simple_3.out
-rw-rw-r-- 1 cogsci50 cogsci50 1.1K Jul 24 13:29 simple-HTC.sub
-rw-rw-r-- 1 cogsci50 cogsci50  388 Jul 24 13:28 simple.sh
~~~

The `-l` option to `ls` gives **l**ongform details for each file and 
the `-h` option will report the 'size' column of the `-l` output 
with **h**uman-readable units (e.g. "K" for kilobytes, etc.).

If files of the same type across all four jobs are the same size, 
then the jobs likely ran the same (all succeeded or all failed), 
but we'll know more by examining some of them.

### The Job Log File

To see what steps HTCondor recorded about our jobs, especially 
since this is a first test (perhaps before we would scale up 
to a larger number), let's first
take a look at a log file that HTCondor created for one of the jobs, and 
which we specified with the `log` line of the submit file.

~~~
$ less simple0.log
~~~
{: .language-bash}
~~~
000 (1191.000.000) 07/24 14:14:32 Job submitted from host: <128.135.158.189:9618?addrs=128.135.158.189-9618+[--1]-9618&noUDP&sock=2276_b9a4_3>
...
001 (1191.000.000) 07/24 14:15:08 Job executing on host: <192.170.227.225:20989?CCBID=192.170.227.251:9714%3faddrs%3d192.170.227.251-9714#109380&addrs=192.170.227.225-20989&noUDP>
...
006 (1191.000.000) 07/24 14:15:17 Image size of job updated: 1
        3  -  MemoryUsage of job (MB)
        2464  -  ResidentSetSize of job (KB)
...
005 (1191.000.000) 07/24 14:15:19 Job terminated.
        (1) Normal termination (return value 0)
                Usr 0 00:00:00, Sys 0 00:00:00  -  Run Remote Usage
                Usr 0 00:00:00, Sys 0 00:00:00  -  Run Local Usage
                Usr 0 00:00:00, Sys 0 00:00:00  -  Total Remote Usage
                Usr 0 00:00:00, Sys 0 00:00:00  -  Total Local Usage
        109  -  Run Bytes Sent By Job
        388  -  Run Bytes Received By Job
        109  -  Total Bytes Sent By Job
        388  -  Total Bytes Received By Job
        Partitionable Resources :    Usage  Request  Allocated
           Cpus                 :                 1          1
           Disk (KB)            :       50     1024 1148453316
           Memory (MB)          :        3        1       1920
...
~~~

Note that we can see an event time for when this job was submitted, 
when it started executing, and when it terminated, as well as an 
update on the "Image size" (and MemoryUsage) of the job.

We can also see that the "Job terminated" with a "Normal termination 
(return value 0)", whereas a non-zero return value would indicate 
that or executable exited with a failure. So, no indications of any 
errors for this job! You can look at the log files of other jobs to 
see that they also completed with a zero return value.

**Testing for Resource Usage**
Additionally, we can see that the bottom of the log file indicates 
that this job used MUCH less Disk (space for the job files) and Memory 
(where the computer stored working, non-file data for the work of 
the executable) on the server where it executed. Even though this job 
was "Allocated" much more of these resources than it requested, we could 
reduce our values for `request_disk` and `request_memory` in future 
submissions of the same executable, to guarantee that more of our jobs 
will find big-enough slots to run on, sooner.

### Examining Job Outputs

We can also view the standard output files (`simple_*.out`) and the 
output files created for each job by the executable (`my*.out`). Beyond the
fact that the log files reported non-zero return values, the fact that 
our `my*.out` files were created (and aren't empty) gives us assurance that 
at least THAT part of our job likely did something useful.

Since the contents are expected to be small (which we 
know from `ls -lh`) and unique per job, we can use 
the `cat` command with a wildcard to view their contents:

~~~
$ cat simple*.out
~~~
{: .language-bash}
~~~
_condor_stderr
_condor_stdout
condor_exec.exe
my0.out
_condor_stderr
_condor_stdout
condor_exec.exe
my1.out
_condor_stderr
_condor_stdout
condor_exec.exe
my2.out
_condor_stderr
_condor_stdout
condor_exec.exe
my3.out
~~~
~~~
$ cat my*.out
~~~
{: .language-bash}
~~~
This is job 0, which ran as osg on mwt2-c061.
This is job 1, which ran as osg on uct2-c500.mwt2.org.
This is job 2, which ran as osg on uct2-c502.mwt2.org.
This is job 3, which ran as osg on n035.
~~~

Remember that our `simple.sh` script would print the results of 
`ls` to the standard output, and would `echo` a statement about 
each job to our `my*.out` files.

**What do the contents of these output files tell you about 
HTCondor's execution of each job and its files?**

### Error Files

If we had seen non-zero return values in any of the log files, 
that would have been the first indication of a potential issue. 
Additionally, some programs print error information to standard 
output, but we checked those files, and all seems fine.

Even so, it can be a good idea to check error files, especially
if they have non-zero size in the `ls -lh` examination. If you 
noticed non-zero sizes for any of your error files, examine their 
contents. For example, some of your error files may include warnings 
specific to OSG Connect's automatic setup for each job:

~~~
$ cat simple_0.err
~~~
{: .language-bash}
~~~
/cvmfs/connect.opensciencegrid.org/modules/lmod/init/bash: line 81: cogsci50: command not found
~~~

If you see the above line for any of your jobs, it shouldn't have 
affected the execution of our simple.sh script, which you would already 
know from the examination of our desired output files.


### That's It!

So, you've had a crash course in submitting HTCondor jobs, 
using just one of the ways to submit multiple jobs. For more 
options to various HTCondor features, make sure to check out 
the [HTCondor User Tutorial](https://agenda.hep.wisc.edu/event/1201/session/4/contribution/5/material/slides/1.pdf), 
[HTCondor Manual](http://research.cs.wisc.edu/htcondor/manual/current/index.html) 
(which can be a bit daunting!), and 
[OSG Connect Helpdesk](https://support.opensciencegrid.org/support/home), where you can 
learn even more about:
- options to commands like `condor_q`, `condor_status`, and `condor_history` for viewing completed jobs
- submit file options, including other ways to communicate variations for *many* jobs
- OSG Connect support for various pre-installes software that your job executables can run
- OSG Connect support for large data (greater than ~100 MB input of 1GB output, per job)
