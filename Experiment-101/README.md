# Experiment-101 : Hello World Application on QNX: VMware Virtual Machine and Raspberry Pi

---

## Objective  
To create, build, and execute a simple “Hello World” program in QNX on two targets:  
1) QNX running inside a VMware virtual machine  
2) QNX running on Raspberry Pi hardware

---

## Hardware Requirements

### For VMware Setup
- Host PC / Laptop (Intel/AMD processor)
- Minimum 8 GB RAM recommended
- Ethernet or Wi-Fi network interface
- VMware Workstation / VMware Player
- QNX Neutrino image configured for x86_64 virtual machine

### For Raspberry Pi Setup
- Raspberry Pi board (Model 3 / 4 recommended)
- MicroSD card with QNX image flashed
- Power adapter for Raspberry Pi
- HDMI monitor and cable
- USB keyboard and mouse
- Ethernet cable or Wi-Fi connection
- Host PC running QNX Momentics IDE

---

## Software / Tools Required  

- QNX Momentics IDE (installed on host machine)
- QNX Neutrino RTOS images for:
  - VMware VM
  - Raspberry Pi
- QNX cross-compiler toolchain (`qcc`)
- Target connection through:
  - Network (preferred) **or**
  - Serial console

---

## Program (hello.c)

```c
#include <stdio.h>

int main(void) {
    printf("Hello World!!!\n");
    printf("Name:Arun\n");
    printf("Regisgter Number:1234567\n");
    return 0;
}
```
## Output
![Output Image](exp1.png)
---

## Result
Thus, a simple Hello World application was successfully developed, compiled, and executed on the QNX operating system using both VMware virtual machine and Raspberry Pi hardware targets.

---

## Additional Program: Process Creation using fork()

### Aim
To create a child process using `fork()` and display the process ID (PID) of the parent and the child.

### Program

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

### Expected Output

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


