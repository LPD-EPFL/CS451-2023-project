# Distributed Algorithms 2023/24 - EPFL


# Overview
The goal of this practical project is to implement certain building blocks necessary for a decentralized system. To this end, some underlying abstractions will be used:

  - Perfect Links (submission #1),
  - Best-effort Broadcast,
  - Uniform Reliable Broadcast,
  - FIFO Broadcast (submission #2),
  - Lattice Agreement (submission #3)

Various applications (e.g., a payment system) can be built upon these lower-level abstractions. We will check your submissions (see [Submissions](#submissions)) for correctness and performance as part of the final evaluation.

For a quick description of the project, refer to [these](https://docs.google.com/presentation/d/14mpRVvEgAYzvws48hUljDygBaFdfGvqqPQP6AJVLLGA/edit?usp=sharing) slides.

For a quick overview of the project's evaluation, refer to [these](https://docs.google.com/presentation/d/1ypP3EFN1Prydo_sBTTdmqOAw_ZwamnB3otyhIb6NHOo/edit?usp=sharing) slides.

All details about the project are given below.

# Project Requirements

## Basics
The implementation must take into account that **messages** exchanged between processes **may be dropped, delayed or reordered by the network**. The execution of processes may be paused for an arbitrary amount of time and resumed later. Processes may also fail by crashing at arbitrary points of their execution.

## Programming Languages
In order to have a fair comparison among implementations, as well as provide support to students, the project must be developed using the following tools:

*Allowed programming language*:
  - C11 and/or C++17
  - Java

*Build system*:
  - CMake for C/C++
  - Maven for Java

Note that we provide you a template for both C/C++ and Java. It is mandatory to use the template in your project. You are **NOT** allowed to change any of the file names or the function signatures in this template.

Allowed 3rd party libraries: **None**. You are not allowed to use any third party libraries in your project. C++17 and Java 11 come with an extensive standard library that can easily satisfy all your needs. However, you allowed to use code from the internet (e.g., a publicly available [skip list](https://en.wikipedia.org/wiki/Skip_list)), as long as you give appropriate credit.

## Communication Primitive
Inter-process point-to-point messages (at the low level) must be carried exclusively by UDP packets in their most basic form, not utilizing any additional features (e.g., any form of feedback about packet delivery) provided by the network stack, the operating system or external libraries. Everything must be implemented on top of these low-level point to point messages.

## Application messages
The messages exchanged by processes contain a payload, which differs based on the submission.
For the first two submission, the payload is an integer number. For the third submission, it is a list of integer numbers.
Apart from the payload, the exchanged messages should also contain other metadata information which is necessary to implement the required abstractions.
We ensure that a messages up to 64KiB can fit in a single UDP packet, thus a sane implementation should not worry about [message fragmentation](https://en.wikipedia.org/wiki/Maximum_transmission_unit).

## Code structure
We provide you a template for both C/C++ and Java, which you should use in your project. The template has a certain structure that is explained below:

### For C/C++:
```bash
.
├── bin
│   ├── deploy
│   │   └── README
│   ├── logs
│   │   └── README
│   └── README
├── build.sh
├── cleanup.sh
├── CMakeLists.txt
├── run.sh
└── src
    ├── CMakeLists.txt
    └── ...
```
You can run:
  - `build.sh` to compile your project
  - `run.sh <arguments>` to run your project
  - `cleanup.sh` to delete the build artifacts. We recommend running this command when submitting your project for evaluation.

You should place your source code under the `src` directory. You are not allowed to edit any files outside the `src` directory. Furthermore, you are not allowed to edit sections of `src/CMakeLists.txt` that are marked as "DO NOT EDIT". Apart from these restrictions, you are completely free on how to structure the source code inside `src`. Yet, we encourage modular design. This will make your code cleaner and the project easier. It will also help you spot bugs.

The template already includes some source code under `src`, that will help you with parsing the arguments provided to the executable (see below).

Finally, **your executable should not create/use directories named "deploy" and/or "logs"** in the current working directory. These directories are reserved for evaluation!

### For Java:
```sh
.
├── bin
│   ├── deploy
│   │   └── README
│   ├── logs
│   │   └── README
│   └── README
├── build.sh
├── cleanup.sh
├── pom.xml
├── run.sh
└── src
    └── main
        └── java
            └── cs451
                └── ...
```
The restrictions for the C/C++ template also apply here. The difference is that you are only allowed to place your source code under `src/main/java/cs451`.

## Application Interface
The templates provided come with a command line interface (CLI) that you should use in your deliverables. The implementation for the CLI is given to you for convenience. You are allowed to make any modifications to it (e.g., add additional argument to help you debug your implementation), as long as it complies to the specification.

The supported arguments are:
```sh
./run.sh --id ID --hosts HOSTS --output OUTPUT CONFIG
```

Where:
  - `ID` specifies the unique identifier of the process. In a system of `n` processes, the identifiers are `1`...`n`.
  - `HOSTS` specifies the path to a file that contains the information about every process in the system, i.e., it describes the system membership. The file contains as many lines as processes in the system. A process identity consists of a numerical process identifier, the IP address or name of the process and the port number on which the process is listening for incoming messages. The entries of each process identity are separated by a single space character. The following is an example of the contents of a `HOSTS` file for a system of 5 processes:
  ```
1 localhost 11001
2 localhost 11002
3 localhost 11003
4 localhost 11004
5 localhost 11005
  ```
  **Note**: The processes should listen for incoming messages in the port range `11000` to `11999` inclusive. Each process should use only 1 port.

  - `OUTPUT` specifies the path to a text file where a process stores its output. The formatting of the output text file depends on the submission. 

- `CONFIG` specifies the path to a file that contains additional information for the experimented abstraction (e.g., how many message to broadcast).

## Process Crashes
We simulate process crashes by relying on Linux's signals.
A process that receives a `SIGTERM` or `SIGINT` signal must immediately stop its execution with the exception of writing to an output log file (described above). In particular, it must not send or handle any received network packets. You can assume that at most a minority (e.g., 1 out of 3; 2 out of 5; 4 out of 10, ...) processes may crash in one execution.
You can assume that a process crash will be simulated only by the `SIGINT` or `SIGTERM` signals.

**Note:** The most straight-forward way of logging the output is to append a line to the output file on every broadcast or delivery event. However, this may harm the performance of the implementation. You may consider more sophisticated logging approaches, such as storing the logs in memory and periodically write them to the output file. Also note that even a crashed process needs to output the sequence of events that occurred before the crash. Remember that writing to files is the only action we allow a process to do after receiving a `SIGINT` or `SIGTERM` signal.

## Applications
### Perfect Links application
In this application, a set of processes exchange messages using perfect links.
In particular a single process only receives messages while the rest of processes only send messages.
The communication of every sender with the receiver is realized using the perfect links abstraction.

Below are the details for the `CONFIG` and `OUTPUT` files of this submission.

#### The `CONFIG` file
  - The config file contains two integers `m i` in its first line. The integers are separated by a single space character. `m` defines how many messages each sender process should send. `i` is the index of the receiver process. The receiver process only receives messages while the sender processes only send messages. All `n-1` sender processes, send `m` messages each.
  Sender processes send messages `1` to `m` in order.

#### The `OUTPUT` file
The output file contains a log of send/receive events. Each event is represented by one line of the output file, terminated by a Unix-style line break `\n`. There are two types of events to be logged:
    - sending of an application message, using the format `b`*`seq_nr`*, where `seq_nr` is the sequence number of the message. These messages are numbered sequentially at each process, starting from `1`.
    - delivery of an application message, using the format `d`*`sender`* *`seq_nr`*, where *`sender`* is the id of the process that sent the message and *`seq_nr`* is the sequence number of the message (as numbered by the sending process).

An example of the content of an output file (ellipsis denotes skipped content)
```
b 1
b 2
b 3
...
```

*Note*: By default, the payload carried by an application message is only the sequence number of that message. Though the payload is known in advance, your implementation should not utilize this information. In other words, your implementation should be agnostic to the contents of the payload. For example, your implementation should work correctly if the payload is arbitrary text instead of sequential numbers. In addition, your implementation should not rely on the fact that the total number of messages (to be broadcast) is known in advance, i.e., your implementation should work correctly if messages arrive as a stream.

### FIFO Broadcast application
This is the same abstraction as the one taught in class. Informally, every process is both broadcasting and delivering messages from every other process (including itself). You must implement FIFO broadcast on top of Uniform Reliable Broadcast (URB).

#### The `CONFIG` file
  - The config file contains an integer `m` in its first line. `m` defines how many messages each process should broadcast. Processes broadcast messages `1` to `m` in order.

#### The `OUTPUT` file
The output file contains a log of broadcast/deliver events. Each event is represented by one line of the output file, terminated by a Unix-style line break `\n`. There are two types of events to be logged:
    -  broadcast of an application message, using the format `b`*`seq_nr`*,
  where `seq_nr` is the sequence number of the message. These messages are numbered sequentially at each process, starting from `1`.
    - delivery of an application message, using the format `d`*`sender`* *`seq_nr`*, where *`sender`* is the number of the process that broadcast the message and *`seq_nr`* is the sequence number of the message (as numbered by the broadcasting process).

An example of the content of an output file (ellipses denote skipped content in this example)
```
b 1
b 2
b 3
...
d 2 1
...
d 4 2
...
b 4
```

### Lattice Agreement
Lattice Agreement is strictly weaker than consensus, as it can be solved in the asynchronous model. The formal specification, as well as an algorithm that implements it, will be provided in moodle. In a nutshell, processes propose sets of values and also decide set of values.
However, notice that the provided algorithm is single-shot, i.e., processes propose and decide only once.

The goal of this submission is to implement multi-shot lattice agreement, in which processes decide on a sequence of proposals.
In other words, in multi-shot lattice agreement, processes run single-shot lattice agreement on a series of slots. The specification of 
lattice agreement must be satisfied individually in every every slot.

#### The `CONFIG` file
  - The config file consists of multiple lines.
  - The first line contains three integers, `p vs ds` (separated by single spaces). `p` denotes the number of proposals for each process, `vs` denotes the maximum number of elements in a proposal, and `ds` denotes the maximum number of distinct elements across all proposals of all processes.
  - The subsequent `p` lines contain proposals. Each proposal is a set of positive integers, written as a list of integers separated by single spaces. Every line can have up to `vs` integers.

An example of the content of a config file (ellipsis denotes skipped content)
```
100 3 5
478163327
958682846 478163327 1181241943
478163327 958682846
107420369
958682846
478163327 107420369 1181241943
...
```

*Note*: In the previous submissions, every process received the same config file. In this submission, every process receives a different config file. These config files have identical first lines and differ in the rest of the lines.

#### The `OUTPUT` file
The text file contains a log of decisions. Each decision is represented by one line of the output file, 
by a Unix-style line break `\n`. Given that proposals are set of integers, so are decisions. Thus, a decision **should** contain a list of integers separated by single spaces (and terminated by a Unix-style line break `\n`). The order of the lines in the output file must be the same as the order of the lines (proposals) in the config file.

An example of the content of an output file (ellipses denote skipped content in this example)
```
478163327 1181241943 1051802512
478163327 958682846 1181241943
478163327 958682846 1181241943
107420369
107420369 958682846
478163327 958682846 107420369 1181241943
...
```

## Compilation
All submitted implementations will be tested using Ubuntu 18.04 running on a 64-bit architecture.
These are the specific versions of toolchains where your project will be tested upon:
  - gcc (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0
  - g++ (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0
  - cmake version 3.10.2
  - OpenJDK Runtime Environment (build 11.0.8+10-post-Ubuntu-0ubuntu118.04.1)
  - Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)

## Project Development
You can develop your project using the following ways:

### Using the provided Virtual Machine (VM)
This is the recommended option.
* It already includes the exact versions of the build tools mentioned previously.
* We provide support for it.
* Thus, students with limited programming knowledge are advised to use it.

Get the VM Image from [here](https://github.com/LPD-EPFL/CS451-2023-project/blob/master/VM.txt) (Windows/Linux/Mac machines with Intel/AMD CPUs, as well as Mac machines with ARM CPUs).

### Using the Docker container
This is recommended only to students with an existing knowledge in distributed programming.
* It already includes the exact versions of the build tools mentioned previously.
* It only works in Linux and Mac
* We do not provide support for it.
* Follow [these](https://github.com/LPD-EPFL/CS451-2023-project/tree/master/docker) instructions to build the container.

### Doing it your way
This is not advised. Do it at your own risk.


**Submissions that fail to compile will NOT be considered for grading. Similarly, submissions that fail to produce any output files or produce faulty output files (e.g., empty files) will NOT be graded.**


## Running the project
We provide two ways of running your project. The first way is manually, which is useful for early development and debugging. The second way is automatic, using the provided python [script](https://github.com/LPD-EPFL/CS451-2023-project/blob/master/tools/stress.py).

### Manual execution

#### Perfect Links application
The following example builds and starts 3 processes (run from within the `template_cpp` or the `template_java` directory):
```sh
# Build the application:
./build.sh

# In first terminal window:
./run.sh --id 1 --hosts ../example/hosts --output ../example/output/1.output ../example/configs/perfect-links.config

# In second terminal window:
./run.sh --id 2 --hosts ../example/hosts --output ../example/output/2.output ../example/configs/perfect-links.config

# In third terminal window:
./run.sh --id 3 --hosts ../example/hosts --output ../example/output/3.output ../example/configs/perfect-links.config

# Wait enough time for all processes to finish processing messages.
# Type Ctrl-C in every terminal window to create the output files.
# Of course, you will NOT find any output files after running this because there is nothing implemented now!
```

#### FIFO Broadcast application
The following example builds and starts 3 processes (run from within the `template_cpp` or the `template_java` directory):
```sh
# Build the application:
./build.sh

# In first terminal window:
./run.sh --id 1 --hosts ../example/hosts --output ../example/output/1.output ../example/configs/fifo-broadcast.config

# In second terminal window:
./run.sh --id 2 --hosts ../example/hosts --output ../example/output/2.output ../example/configs/fifo-broadcast.config

# In third terminal window:
./run.sh --id 3 --hosts ../example/hosts --output ../example/output/3.output ../example/configs/fifo-broadcast.config

# Wait enough time for all processes to finish processing messages.
# Type Ctrl-C in every terminal window to create the output files.
```

#### Lattice Agreement
The following example builds and starts 3 processes (run from within the `template_cpp` or the `template_java` directory):
```sh
# Build the application:
./build.sh

# In first terminal window:
./run.sh --id 1 --hosts ../example/hosts --output ../example/output/1.output ../example/configs/lattice-agreement-1.config

# In second terminal window:
./run.sh --id 2 --hosts ../example/hosts --output ../example/output/2.output ../example/configs/lattice-agreement-2.config

# In third terminal window:
./run.sh --id 3 --hosts ../example/hosts --output ../example/output/3.output ../example/configs/lattice-agreement-3.config

# Wait enough time for all processes to finish processing messages.
# Type Ctrl-C in every terminal window to create the output files.
```

### Automatic execution
Depending on the submission, use the appropriate command below:
```sh
./stress.py perfect -r RUNSCRIPT -l LOGSDIR -p PROCESSES -m MESSAGES
./stress.py fifo -r RUNSCRIPT -l LOGSDIR -p PROCESSES -m MESSAGES
./stress.py agreement -r RUNSCRIPT -l LOGSDIR -p PROCESSES -n PROPOSALS -v PROPOSAL_MAX_VALUES -d PROPOSALS_DISTINCT_VALUES

```

Where:
* `RUNSCRIPT` is the path to *run.sh*. Remember to build your project first!
* `LOGSDIR` is the path to a directory where stdout/stderr and output of each process will be stored.  It also stores generated HOSTS and CONFIG files. The directory must exist as it will not be created for you.
* `PROCESSES` (for `perfect`, `fifo` and `agreement`) specifies the number of processes spawn during validation.
* `MESSAGES` (for `perfect` and `fifo`) specifies the number of messages each process is broadcasting.
* `PROPOSALS` (for `agreement`) specifies the number of proposals each process is proposing.
* `PROPOSAL_MAX_VALUES` (for `agreement`) specifies the maximum size of the proposal set, i.e., the maximum number of integers that each proposal contains.
* `PROPOSALS_DISTINCT_VALUES` (for `agreement`) specifies the number of distinct elements across all proposals of all processes, i.e., it specifies the maximum size of the union of all proposals.

You can edit `testConfig` at the bottom of `stress.py` in order to test different scenarios.
```py
testConfig = {
  'concurrency' : 8, # How many threads are interfering with the running processes.
  'attempts' : 8, # How many successful operations (SIGCONT, SIGSTOP, SIGTERM) each thread will attempt before stopping. Threads stop if a minority of processes has been terminated.
  'attemptsDistribution' : { # Each thread selects a process randomly and issues one of these operations with the given probability.
    'STOP': 0.48,
    'CONT': 0.48,
    'TERM':0.04
    }
}
```

To change the rate at which interfering threads interfere with the processes modify this line: 
```py
time.sleep(float(random.randint(50, 500)) / 1000.0)
```
under the `StressTest` class.


## Limits
The entire project implements abstractions that operate in the asynchronous model, i.e., there is no bound in processing and communication delays. However, during the evaluation of the projects we set a maximum execution time limit.

In particular, we assume that every broadcast message takes at most 1 second to be delivered when:
* the network is not delaying/dropping/reordering packets.
* Processes are not delayed (e.g., with SIGSTOP).

We adjust this number accordingly when the network is not well behaving.

Additionally, we enforce certain limitations during testing:
* You are given 2 CPU cores.
* You are given a maximum of 64MiB per process.
* You are allowed to use a maximum of 8 threads per process.

Poorly engineered implementations may fail some tests due to these limitations (e.g., if you create a new thread for every message you broadcast).

Finally, two more limitation are in place:
* Your output files cannot exceed 16MiB.
* Your console output files are trimmed if they exceed 1MiB.

These are only to guard against programming mistakes (e.g., infinite loop in the output that depletes disk space). You should not be worried about them.

## Cooperation
This project is meant to be completed individually. Copying from others is prohibited. You are free (and encouraged) to discuss the projects with others, but the submitted source code must be your exclusive work. Multiple copies of the same code will be disregarded without investigating which is the "original" and which is the "copy". Furthermore, please give appropriate credit to pieces of code you found online (e.g. in stackoverflow).

*Note*: code similarity tools will be used to check copying.

## Submissions and Grading
### Grading
This project accounts for 30% of the final grade and comprises three submissions:
  - A runnable application implementing Perfect Links (20%),
  - A runnable application implementing FIFO Broadcast (40%), and
  - A runnable application implementing Lattice Agreement (40%).

Note that these submissions are *incremental*. This means that your work towards the first will help you in your work towards the second.

### Deadlines
* Perfect Links: November 5th 2023, 23:59
* Fifo Broadcast: December 3rd 2023, 23:59
* Lattice Agreement: December 24th 2023, 23:59

### Submissions
Submit your deliverable by uploading it to [this](https://cs451-submissions.epfl.ch:8083) website. Use EPFL’s VPN if you cannot access the submission website.

The website stores the submission, tests it and then provides with feedback.
We advise you to submit regularly, in order to avoid “silly” mistakes (e.g., hardcoding the OUTPUT path).
Do not use the website to develop (e.g., submit 100 times a day) your project!

**These deadlines mentioned above are definitive. The website does not accept new submissions past the specified deadline.**

**We do not accept submissions over email. Submissions sent over email will not be considered.**

Students registered to ISA-Academia and/or Moodle will receive by email a passphrase to access the website.
Read more about how to upload your deliverable to the submission website [here](https://docs.google.com/document/d/1Ai3tQeaTLD0p_2HrVTlONOUskoEExj1HdVOlfIbHdXQ).

The website runs a simple test to evaluate your submission. Further tests will be ran after the submission deadline.

**Only submissions that pass the website test will be graded.**

If your submission passes the initial validation, we will evaluate it based on two criteria: correctness and performance. We prioritize correctness: a correct implementation (i.e., that passes all the test cases) will receive (at least) a score of 4-out-of-6. The rest 2-out-of-6 is given based on the perfomance of your implementation compared to the perfomance of the implemantions submitted by your colleagues. The fastest correct implementation will receive a perfect score (6). Incorrect implementations receive a score below 4, depending on the number of tests they fail to pass.

# FAQ
**1. Can I use multi-threading?**

Yes! However, only spawn threads (don't spawn child ​processes​) from your process.
C11/C++17 and Java support threads natively.

**2. Can I put multiple messages in the same packet?**

Yes, but we limit the number of messages per same packet to 8. Moreover, you should not use the fact that the payloads are sequential integers nor that the total number of messages is known in advance. For example, your code should work correctly if the payload is some arbitrary text.

**3. Can I compress the messages?**

Yes. This is an implementation detail that is up to you.

**4. I implemented the whole project but it does not work correctly (or does not compile). Will I get some points for the implementation?**

No. Submissions that fail to compile will NOT be considered for grading. Similarly, submissions that fail to produce any output files or produce faulty output files (e.g., empty files) will NOT be graded.

**5. Which performance should I aim for? How many messages per second?**

You should aim for maximum performance. You can assume that the number of messages will not be more than MAX_INT, i.e., each process will not broadcast more than 2147483647 messages. Also, you can assume that the broadcasting processes will be no more than 128. We know that there are always hardware limits so do not worry too much about this issue. These limits are given to help you with the message structure (e.g., 1 byte is enough to encode the process identifier).

**6. Will there be separate performance rankings for C/C++ and Java?**

Yes. C/C++ and Java will be graded separately since the performance may be drastically different. It is up to you to choose the language you are the most comfortable with. We will calibrate the performance of both languages.

**7. How can I introduce delay/loss/reordering in the network?**

For the purposes of this project all processes execute on the same machine and use the loopback interface.
You can easily use [TC](https://man7.org/linux/man-pages/man8/tc.8.html) to change the characteristics of this interface and introduce the desired delay/loss/reordering.
We provide the `tools/tc.py` script to simplify the process, however we urge you to consider changing the parameters inside this script to gain confidence in the correctness of your implementation.
Run this script before starting the processes and keep it running for the duration of the execution.

**8. How can I validate my output?**

It is your responsibility to ensure that your implementation is correct.
However, we provide a sample validation script for the FIFO broacast (`tools/validate_fifo.py`). This script uses the output files generated by the processes after they terminate their execution.

**9. Is it ok that the processes terminate before they are able to deliver all the messages?**

Yes, as soon as you receive a SIGTERM signal, you need to terminate the process and start writing to the logs. You may not have delivered all the messages by that time which is ok. You should only deliver the message that you can deliver. i.e., that does not violate FIFO and URB. If instead you do, while you are not allowed to, you may be violating correctness.

**10. Which files can I modify in the template?**

In general, you can add whatever you want to ANY file in the template that is NOT marked with DO NOT EDIT. Also, you are NOT allowed to change the function signatures nor the file names. Other than that (like adding a parser, adding some code in the main, create new classes, adding new functions, adding constants, ...etc.) is allowed.
