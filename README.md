# CampusCompute – What we Ran and Why

**SRN:** PES1UG25CS193,PES1UG25CS191(PES1UG25CS193 is visible in the screenshots)
**Environment:** Ubuntu on VirtualBox (GCC 15.2.0, Clang 21.1.8, GNU Make 4.4.1)



## 1. Getting set up

### S01 – Environment and SRN prompt

```bash
unzip CampusCompute.zip      
cd CampusCompute
pwd
export SRN='PES1UG25CS193'
export PS1='[$SRN] \w\$ '
```

we started by unzipping the project. It asked whether to replace files that were already there, so we answered `y` each time. Then we moved into the folder with `cd` and ran `pwd` to make sure we were in `/home/niharikag/Downloads/CampusCompute`.

The last two lines set member 1's SRN as a variable and put it in the shell prompt. That's why `[PES1UG25CS193]` shows up at the start of every line in the screenshots.

### S02 – Compiler and build tools

```bash
sudo apt update
sudo apt install build-essential clang make unzip -y
gcc --version
clang --version
make --version
```

Before building anything we wanted to be sure the tools were there. `apt update` refreshed the package list, and the install command checked for GCC, Clang, Make and Unzip. Everything was already installed and up to date, so nothing new was downloaded. The three `--version` commands confirmed GCC 15.2.0, Clang 21.1.8 and Make 4.4.1.

## 2. Looking at the project

### S03 – Project files

```bash
ls -la
find . -maxdepth 2 -type f
```

`ls -la` lists everything in the folder, including hidden files, along with sizes and permissions. we could see all the source files (`main.c`, `scheduler.c`, `advisor.c`, `banker.c`, `worker.c`, `logger.c`, `ipc.c`, `limits.c`), their headers, the tests, the `Makefile`, the README, and the `member1`, `member2`, `logs`, `tests` and `screenshots` folders.

`find . -maxdepth 2 -type f` gave us the same picture one level deeper, so we could check that the files inside the subfolders were all there too.

## 3. Building it

### S04 – Clean build

```bash
make clean
make CC=gcc
ls -l campuscompute
```

we cleaned first so I'd be building from scratch. `make clean` deletes the old executable, the test binaries and the log file.

`make CC=gcc` then compiled everything with:

```
gcc -Wall -Wextra -std=c11 main.c scheduler.c advisor.c banker.c worker.c logger.c ipc.c limits.c -o campuscompute -pthread -lm
```

`-Wall -Wextra` turns on extra warnings, `-std=c11` uses the C11 standard, `-pthread` is needed for the logger's threads, and `-lm` links the math library. It compiled with no warnings or errors, and `ls -l campuscompute` showed the new 43,776-byte executable.

## 4. Running the main program

### S05 – Main menu

```bash
./campuscompute
```

This starts the program. It prints the banner with member 1's SRN and the menu: CPU Scheduling, Workload Intelligence, Banker's Algorithm, Worker Management, Inter-Process Communication, Concurrent Logger, Resource Limits, Run All Modules, and Exit.

### S06 – FCFS scheduling

we picked option `1`. The program uses three processes: P1 (arrival 0, burst 5, priority 2), P2 (arrival 1, burst 3, priority 1) and P3 (arrival 2, burst 1, priority 3).

FCFS ran them in order: `P1 0-5 | P2 5-8 | P3 8-9`. The averages were 3.33 for waiting time, 6.33 for turnaround and 3.33 for response, with everything finishing at time 9.

### S07 – Round Robin

Round Robin ran straight after FCFS. The timeline was `P1 0-2 | P2 2-4 | P3 4-5 | P1 5-7 | P2 7-8 | P1 8-9`. Waiting and turnaround averages stayed at 3.33 and 6.33, but the average response time dropped to 1.00, since every process gets the CPU sooner.

### S08 – Priority scheduling

Priority scheduling came next and produced the same timeline as FCFS (`P1 0-5 | P2 5-8 | P3 8-9`) with the same averages. After it, the Workload Intelligence module looked at the job mix, called it priority-sensitive, and recommended Round Robin.

### S09 – Comparing the algorithms

This screenshot puts all three side by side. Waiting and turnaround times came out identical for this input. The only real difference is response time, where Round Robin wins at 1.00 compared with 3.33, and that's why the program recommends it.

## 5. Running the module tests

 went into the `member2` folder and ran each test program on its own:

```bash
cd member2
./test_logger
./test_ipc
./test_limits
./test_banker
./test_worker
```



### S10 – Producer-consumer logger

```bash
./test_logger
```

This starts 4 producer threads that each write 5 events, so 20 in total. The `[LOGGER]` lines show the events coming in with timestamps. At the end it says the queue was flushed and prints **CONCURRENT LOGGER TEST PASSED**, so all 20 events made it through.

### S11 – Thread synchronization and termination

```bash
./test_logger
./test_ipc
```

The line "All producer threads completed successfully" shows the threads finished cleanly. No events were lost or duplicated, which is the point of using locks and a shared queue. `test_ipc` ran right after and passed as well.

### S12 – Shared-memory IPC

```bash
./test_ipc
```

The parent creates a shared memory segment and a semaphore. The child process writes a message into the shared memory, and the parent reads it back ("Worker report: CampusCompute job completed."). The test ends with **IPC TEST PASSED**.

### S13 – Semaphore coordination and cleanup

```bash
./test_ipc
```

Same program, but here we are pointing out two lines. "Shared memory and semaphore created" appears at the start, and "cleanup completed" at the end. The semaphore makes sure the parent only reads after the child has finished writing, and the cleanup line shows nothing is left behind in the system.

### S14 – Banker's algorithm, safe state

```bash
./test_banker
```

Test 1 uses 5 processes and 3 resource types, with 3 3 2 resources available. The need matrix (Max minus Allocation) is P0: 7 4 3, P1: 1 2 2, P2: 6 0 0, P3: 0 1 1, P4: 4 3 1. The program walks through which process can finish and releases its resources each time, and finds the safe sequence **P1 → P3 → P4 → P0 → P2**. Result: safe state, TEST 1 PASSED.

### S15 – Banker's algorithm, unsafe state

Test 2 starts with 0 0 0 resources available, so no process can ever get what it needs. The program reports an unsafe state, says no safe sequence exists, and lists P0 to P4 as unable to finish. TEST 2 PASSED, followed by "All Banker tests passed."

### S16 – Normal worker process

```bash
./test_worker
```

Test 1 forks a child process (PID 6473). It finishes its job and exits with code 0, and the parent confirms it with "Worker completed normally." TEST 1 PASSED.

### S17 – Simulated worker failure

Test 2 forks another child (PID 6474) that deliberately fails and exits with code 2. The parent notices the non-zero exit code and prints "Worker failure detected." TEST 2 PASSED, and then "All worker tests passed."

### S18 – Resource limit

```bash
./test_limits
```

The child sets a CPU time limit of 1 second and then keeps running until the limit is hit. The system kills it ("Child terminated by signal 9"), and the parent prints that the demonstration is complete. This shows resource limits really are enforced.

## 6. Checking bad input and exiting

### S19 – Invalid input

```bash
cd ..
./campuscompute
```

we went back to the main folder and started the program again. At the prompt we typed `gg` instead of a number. It printed "Invalid input." and showed the menu again, so it handles bad input without crashing.

### S20 – Clean exit

we entered `0`. The program printed "Thank you for using CampusCompute!" and returned me to the normal shell prompt.

## 7. Full run and logs

### S21 – Integrated execution

```bash
./campuscompute
```

At the prompt we chose `8` (Run All Modules). It ran everything in order: the three scheduling algorithms, Workload Intelligence, Banker's algorithm (same safe sequence as before), worker management, IPC, the concurrent logger and the resource-limit demo. Then it returned to the menu.

### S22 – Generated logs

```bash
cat campuscompute.log
cat member2/campuscompute.log
```

`cat` just prints a file. The first log has the entries from the integrated run, such as "CampusCompute integrated application started". The second has the logger test output with timestamped "Producer N: event M" lines from several different runs.

## 8. Final check and packaging

### S23 – Final files

```bash
ls -l
```

One last listing of the project folder to show everything is in place: sources, headers, tests, `Makefile`, README, the built `campuscompute` executable, the log file, and the subfolders.

### S24 – Submission ZIP

```bash
cd ~/Downloads
zip -r CampusCompute_Submission.zip CampusCompute    
unzip -t CampusCompute_Submission.zip
unzip -l CampusCompute_Submission.zip
```

we zipped the project folder, then tested the archive with `unzip -t`. Every file came back `OK` with "No errors detected in compressed data". Finally `unzip -l` listed what's inside, so we could confirm the sources, logs and member folders all made it in.

##TEAM MEMBERS CONTRIBUTIONS

Goutami velaga,PES1UG25CS191

Implementation and Development

 Worked on developing the main parts of the project.
 Implemented the FCFS, Round Robin, and Priority Scheduling algorithms.
 Developed the producer-consumer logging mechanism using threads.
 Worked on thread synchronization and termination.
 Implemented shared-memory communication and semaphore operations.
 Developed the Banker's Algorithm to check safe and unsafe states.
 Combined the different modules to make the project work as a complete system.

 GUDDEMMA GARI NIHARIKA,PES1UG25CS193

 Testing and Validation

  Tested the different modules to check whether they were working correctly.
  Tested all three CPU scheduling algorithms with different inputs.
  Checked the scheduling results and compared the algorithms.
  Tested the producer-consumer mechanism and thread synchronization.
  Verified shared-memory communication and semaphore operations.
  Tested the Banker's Algorithm for both safe and unsafe states.
  Checked how the program handles worker failures and resource limits.
  Tested invalid inputs and verified that cleanup operations worked properly.
  Ran the complete project to ensure all the modules worked together as expected.


##GITHUB url
https://github.com/goutamivelaga77/CampusCompute.git






