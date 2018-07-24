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
my2.out  simple_0.log  simple_1.log  simple_2.log  simple_3.log  simple-job.sh
~~~

That's quite a few files, so let's look at a few of them at a time. 
Taking a look at our submit file and executable script will help us remember 
what's what:

~~~
$ cat *.sub
~~~
{: .language-bash}
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
# We'll estimate relatively small amounts of memory and disk space for each job,
#  and one CPU (the default):
request_cpus = 1
request_memory = 1MB
request_disk = 1MB
#
# Tell HTCondor to run 4 instances of our job:
queue 4
~~~
~~~
$ cat *.sh
~~~
{: .language-bash}
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

**So the files we should see include:**

Setup files:
- `simple-job.sh`, our executable
- `simple-HTC.sub`, our submit file queueing four jobs using the executable

"metadata" files specified in the submit file for each of our jobs (numbered by job 'process' value):
- `simple_*.out`, where the terminal or "standard" output from the executable is saved
- `simple_*.err`, where (standard) error information from the executable is saved
- `simple_*.log`, where HTCondor has recorded the steps of running the job for us

And a single output file produced by our executable, for each job:
- `my*.out`, which was created by our executable

