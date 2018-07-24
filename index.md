---
layout: lesson
root: .
---

This lesson guides you through the basics of submitting [HTCondor](https://research.cs.wisc.edu/htcondor/) 
jobs on the [OSG Connect service](https://osgconnect.net/), including a 
neuroscience-specific example of high-throughput computing from Chris Cox's own 
research. HTCondor is software installed across the Open Science Grid (and many other 
computing systems, "pools", or "clusters" around the world) that can schedule 
large numbers of computing "jobs" to run across large numbers of disparate, heterogeneous 
computers.

While some of the neuroscience-specific example leverages Matlab versions available 
through OSG Connect, the material about interacting with HTCondor would work on 
any HTCondor computing system.

> ## Prerequisites
>
> Prior experience with the Unix Shell (bash) is necessary, including skills 
> covered in the morning portion of the tutorial, such as navigating a unix 
> file system using absolute and relative paths, creating and editing text files 
> on the command line, using wildcards, and creating shell scripts to automate 
> the execution of several commands, in order. While there is some Matlab and Python 
> code used in the neuroscience-specific example, you do not need to know how to 
> program in these languages in order to submit the example jobs that use them.
