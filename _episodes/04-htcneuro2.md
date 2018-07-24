---
title: "Setting Up a Neuroscience HTC Workload 2: setupJobs and stub.yaml"
teaching: 10
exercises: 0
questions:
- "How does setupJobs turn a stub.yaml into a populated directory tree?"
objectives:
- "Unpack the stub.yaml."
- "Review the features of setupJobs."
keypoints:
- "The setupJobs workflow can is not project specific. The parameters listed in the stub file can be tailored to any needs."
- "EXPAND, COPY, and URL fields in stub.yaml define 3 operations that can be applied to (combinations of) parameter field listed elsewhere in the stub.yaml file."
---

## Overview

Now that you have worked through one preset example, the goal of this section is to show how InputSetup and specifically `setupJobs` can be used to facilitate any arbitrary HTC analysis.

## The simplest example: No special fields

The stub.yaml we worked with previously contained many parameters that were specific to the analysis we wanted to run and the WISC MVPA workflow.
However, the YAML + `setupJobs` workflow is not specific to WISC MVPA or those parameters.
Let's boil this down and consider the simplest example.

~~~
# simple-stub.yaml
A: 1
B: 2
C: [1,2]
D: [3,4,5]
~~~
{: .language-yaml}

In this file, I define 4 parameters.
Two of them are scalar values, and 2 are lists.

Let's run `setupJobs`.

~~~
$ setupJobs stub.yaml
$ find ./ -type f -name "params.json" -print
~~~
{: .language-bash}

~~~
./0/params.json
~~~
{: .output}

~~~
{
    "A": 1,
    "B": 2,
    "C": [ 1, 2 ],
    "D": [ 3, 4, 5 ],
    "SearchWithHyperband": false
}
~~~
{: .language-json}

Nothing interesting here: we basically translated YAML to JSON and added one field that we can ignore.

## The EXPAND field

Now, let's modify `simple-stub.yaml` slightly:

~~~
# simple-stub.yaml
A: 1
B: 2
C: [1,2]
D: [3,4,5]
EXPAND:
    - C
    - D
~~~
{: .language-yaml}

**EXPAND** is a special field.
It will "cross" the lists in C and D, creating one parameter file where B:1 and C:3, another where B:1 and C:4, and so on.
Let's run `setupJobs` again and just see how it goes:

~~~
$ setupJobs stub.yaml
$ find ./ -type f -name "params.json" -print | xargs grep "C\|D"
~~~
{: .language-bash}

~~~
./0/params.json:    "C": 1,
./0/params.json:    "D": 3,
./1/params.json:    "C": 2,
./1/params.json:    "D": 3,
./2/params.json:    "C": 1,
./2/params.json:    "D": 4,
./3/params.json:    "C": 2,
./3/params.json:    "D": 4,
./4/params.json:    "C": 1,
./4/params.json:    "D": 5,
./5/params.json:    "C": 2,
./5/params.json:    "D": 5,
~~~
{: .output}

EXPAND can cross as many fields as you like.
In this way, a single stub file can contain the instructions for building many parameter files.

## COPY field
There are two additional special fields that help with organizing a workload and managing file transfers.
First, let's intoduce **COPY**.
Try the following at the bash shell:

~~~
$ touch cows.txt ducks.txt pigs.txt
~~~
{: .language-bash}

And now we will again slightly revise `simple-stub.yaml`:

~~~
# simple-stub.yaml
A: 'cows.txt'
B: 2
C: ['ducks.txt','pigs.txt']
D: [3,4,5]
EXPAND:
    - C
    - D
COPY:
    - A
    - C
~~~
{: .language-yaml}

~~~
$ setupJobs stub.yaml
$ find . -mindepth 2 -type f -name "cows.txt" -print
~~~
{: .language-bash}

~~~
./shared/cows.txt
~~~
{: .output}

~~~
$ find . -mindepth 2 -type f -name "ducks.txt" -print
~~~
{: .language-bash}

~~~
./0/ducks.txt
./2/ducks.txt
./4/ducks.txt
~~~
{: .output}

~~~
$ find . -mindepth 2 -type f -name "pigs.txt" -print
~~~
{: .language-bash}

~~~
./1/pigs.txt
./3/pigs.txt
./5/pigs.txt
~~~
{: .output}

Notice that `ducks.txt` and `pigs.txt` are copied into different individual directories, while `cows.txt` is copied only into the shared directory.
Since A is constant over jobs, that means every job needs `cows.txt`.
The whole shared folder is sent to every job, so there is no reason to make 6 copies of `cows.txt`.

## URLS field
The final special field is **URLS**.
This field controls what gets put into the `querey_input.csv` file.
You'll notice that up until this point, this file was not being generated.
Once this field is included in `simple-stub.yaml`, it will be.

~~~
# simple-stub.yaml
A: 'cows.txt'
B: 'http://host.org/metadata.mat'
C: ['ducks.txt','pigs.txt']
D: ['http://host.org/data1.mat','http://host.org/data2.mat','http://host.org/data3.mat']
EXPAND:
    - C
    - D
COPY:
    - A
    - C
URLS:
    - B
    - D
~~~
{: .language-yaml}

~~~
$ setupJobs stub.yaml
$ cat queue_input.csv
~~~
{: .language-bash}

~~~
./0,http://host.org/metadata.mat,http://host.org/data1.mat
./1,http://host.org/metadata.mat,http://host.org/data1.mat
./2,http://host.org/metadata.mat,http://host.org/data2.mat
./3,http://host.org/metadata.mat,http://host.org/data2.mat
./4,http://host.org/metadata.mat,http://host.org/data3.mat
./5,http://host.org/metadata.mat,http://host.org/data3.mat
~~~
{: .output}

## Conclusion

The combination of `setupJobs` and the YAML file is a flexible and efficient way to compose multiple jobs by specifying a single file.
Let's now open the stub.yaml file from the previous step.
It should now be a bit more clear how we ended up with 400 jobs by parsing that single file.

There are some features of `setupJobs` that were not reviewed today that my be of interest.
One is that there is special functionality for constructing a hyperparameter search using the [Hyperband](https://homes.cs.washington.edu/~jamieson/hyperband.html) procedure ([paper](https://arxiv.org/abs/1603.06560)).
The usage is exemplified in the `stub.yaml` file used to launch the workload in the previous step, and will very soon be documented at [github.com/crcox/InputSetup](https://github.com/crcox/InputSetup).
Feel free to direct questions to [Chris Cox](mailto:chriscox@lsu.edu) in the meantime.

