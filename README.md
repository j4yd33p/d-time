# D-TIME
__D-TIME: Distributed Threadless Independent Malware Execution for Runtime Obfuscation__

An important aspect of malware design is to be able to evade detection. This is increasingly difficult to achieve with powerful runtime detection techniques based on behavioural and heuristic analysis. In this paper, we propose D-TIME, a new distributed threadless independent malware execution framework to evade runtime detection.

D-TIME splits a malware executable into small chunks of instructions and executes one chunk at a time in the context of an infected thread. It uses a Microsoft Windows feature called Asynchronous Procedure Call (APC) to facilitate chunk invocation; shared memory to coordinate between chunk executions; and a novel Semaphore based Covert Broadcasting Channel (SCBC) for communication between various chunk executions. The small size of the chunks along with the asynchronous nature of the execution makes runtime detection difficult, while the coordinated execution of the chunks leads to the intended malign action. D-TIME is designed to be self-regenerating ensuring high resilience of the system.

We evaluate D-TIME on a Microsoft Windows system with six different malware and demonstrate its undetectability with 10 different anti-virus software. We also study the CPU usage and its influence on Performance Counters.


### Directory and details

| Directory  |    Content                                                                                     |
|------------|:-----------------------------------------------------------------------------------------------|
| PoCs       | Independent PoCs for major concepts used (SCBC and Re-generating APC based Emulators)          |
| emulator   | Sample code for emulator and an example injection technique                                    |
| samples    | Provide 6 malware samples to test D-TIME in your environment                                   |
| splitter   | The code for IDA-Pro plugin which will split the malware into chunks                           |

Description of each module is given in the README of the respective directory.


### How to use

> **DISCLAIMER: All the content provided on this site are for educational purposes only. The site is no way responsible for any misuse of the information**

__It is assumed that the reader has already gone through the research paper - _"D-TIME: Distributed Threadless Independent Malware Execution for Runtime Obfuscation"_  (to be) published in WOOT'19. Understanding of this paper is crucial to understand the following steps.__

Detailed instructions for each of the following steps are given in README of relevant directories or respective files. 

| |
|:-:|
| __IMPORTANT:__ We have tested the D-TIME in __Windows 10 Pro (1803)__ using __Visual Studio 2010__. You are recommended to use a similar setup for your first attempt to avoid confusions. (You can safely use the latest version of Windows 10) |
| |

__Note 1:__ We recommend building the system in debug  mode for the first time. Once you have understood the framework you can build the same in release mode.

__Note 2:__ We recommend attempting the _Offline Keylogger_ sample for the first time. It is the simplest sample that we have provided.


#### Step 1: Offline Phase  
In Offline Phase, we create chunks that will be distributed across threads in the Online Phase. For this,
   1. Build one of the sample malware.
   1. Now we can use the malware binary(output of above build operation) to create malware chunks using `splitter`.
      `splitter` creates the chunks and writes them to separate files.
   1. Follow the instructions provided under `splitter` to generate these files.
      
#### Step 2: Online Phase
In the Online Phase, we inject the emulator to threads and execute malware chunks in a distributed fashion. `emulator` contains instructions to build the emulator along with a sample injector which will inject the emulators for you.
   1. Go to `d_time.h` and correct the following variables:
       1. Uncomment `#define __LOG__`. This preprocessor variable logs the progress in `blkLog.txt` under temporary folder.
       Note that logging is drastically slow down D-TIME. For example, keylogger may not be able to capture all the keystrokes when logging is ON.
       1. Under `#ifndef __SELF_INJECTION__` section, uncomment the applications you like to inject D-TIME into. Instructions regarding victim applications are given in the next section.
   1. Go to `main.cpp` and make the following changes:
       1. `#define _NBLOCKS` and `#define _NSEGMS` should be set to the number of chunks and segments generated by IDA Pro Plugin. The plugin will log these numbers in IDA Pro Console at the end of execution.
       1.  `#define _NPROC` is the number of victims. This should match the number of applications uncommented under `#ifndef __SELF_INJECTION__`.
   1. Build the `emulator`.
   2. Copy the chunk files generated in step 1 to your working directory of emulator.
   1. Make sure that your victim processes are running.
   3. Run emulator.exe.  
   The emulator build contains the  actual emulator code and a sample injector. It will:
       1. Read your chunks from the working directory and store them in shared memory.
       2. Inject the emulator to victim processes.
       3. Exit.  
       The injected emulator will now execute the chunks and re-generate themselves to execute more chunks.

### Victim Processes
| |
|:-:|
| __IMPORTANT:__ D-TIME is a 32 bit applications and requires 32 bit victim processes. |
| |

We have tested the following applications as victims:

1. Chrome (version 74.0.3729.131)
1. Skype (version 8.45.0.41)
1. Opera (version
58.0.3135.132)
1. Acrobat Reader (version 19.10.20099.322322)
1. VLC (version 3.0.6)
1. Calculator (version 1803) (Calculator that comes with Windows 10 is 64 bit. Please use the calculator that comes with 32 bit Windows 7)

Though we have provided the versions we tested on, you can safely use the latest version of these software.
