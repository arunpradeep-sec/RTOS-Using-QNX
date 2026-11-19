# Experiment-101 : Process Creation using fork() in QNX

---

## Objective  
To create, build, and execute a C program in QNX that creates a child process using `fork()` and displays the process ID (PID) of the parent and child processes, on QNX running inside a VMware virtual machine.

---

## Hardware Requirements

### For VMware Setup
- Host PC / Laptop (Intel/AMD processor)
- Minimum 8 GB RAM recommended
- Ethernet or Wi-Fi network interface
- VMware Workstation / VMware Player
- QNX Neutrino image configured for x86_64 virtual machine

---

## Software / Tools Required  

- QNX Momentics IDE (installed on host machine)
- QNX Neutrino RTOS image for the VMware VM
- QNX cross-compiler toolchain (`qcc`)
- Target connection through:
  - Network (preferred) **or**
  - Serial console

---

## Algorithm

1. Start the program.
2. Print **"Program started"**.
3. Call `fork()` to create a new process and store the returned value in `pid`.
4. If `pid` is **0**, it is the **child process**:
   * Print **"I am the Child Process"**.
   * Print the child PID using `getpid()`.
   * Print the parent PID using `getppid()`.
   * Sleep for **30 seconds**.
5. If `pid` is **greater than 0**, it is the **parent process**:
   * Print **"I am the Parent Process"**.
   * Print the parent PID using `getpid()`.
   * Print the child PID (the value returned by `fork()`).
   * Sleep for **30 seconds**.
6. If `pid` is **negative**, process creation has failed:
   * Print **"Process creation failed"**.
   * Terminate the program with return value 1.
7. Return 0 and end the program.

---

## Program

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main(void)
{
    pid_t pid;

    printf("Program started\n");

    pid = fork();

    if (pid == 0)
    {
        printf("I am the Child Process\n");
        printf("Child PID  = %d\n", (int)getpid());
        printf("Parent PID = %d\n", (int)getppid());
        sleep(30);
    }
    else if (pid > 0)
    {
        printf("I am the Parent Process\n");
        printf("Parent PID = %d\n", (int)getpid());
        printf("Child PID  = %d\n", (int)pid);
        sleep(30);
    }
    else
    {
        printf("Process creation failed\n");
        return 1;
    }

    return 0;
}
```

---

## Sample Output

```text
Program started
I am the Parent Process
Parent PID = 265
Child PID  = 266
I am the Child Process
Child PID  = 266
Parent PID = 265
```

*(PID values differ on every run, and the parent and child lines may appear in either order. Both processes sleep for 30 seconds before exiting.)*

---

## Output

<!-- Student: paste your output screenshot here, e.g. ![Output Image](output.png) -->

<br>
<br>
<br>
<br>

---

## Result
Thus, a C program using `fork()` was successfully developed, compiled, and executed on the QNX operating system using a VMware virtual machine, and the PIDs of the parent and child processes were displayed.

---
