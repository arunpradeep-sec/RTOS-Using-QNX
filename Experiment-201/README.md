# Experiment-201 : Process Creation using posix_spawn() in QNX

---

## Objective  
To create, build, and execute a parent program and a child program in QNX, where the parent creates a new child process using `posix_spawn()`, waits for it to finish using `waitpid()`, and both processes display their process ID (PID), on QNX running inside a VMware virtual machine.

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

### Child Program

1. Start the child program.
2. Print **"I am the Child Process"**.
3. Print the child PID using `getpid()`.
4. Print the parent PID using `getppid()`.
5. Return 0 and end the program.

### Parent Program

1. Start the parent program.
2. Declare the variable `pid` and the argument list `{"child", NULL}`.
3. Print **"I am the Parent Process"**.
4. Print the parent PID using `getpid()`.
5. Call `posix_spawn()` with the path `./child`, the argument list and the environment (`environ`) to create the child process. The PID of the new process is stored in `pid` and the return value is stored in `status`.
6. If `status` is **0**, the child was created successfully:
   * Print the child PID.
   * Wait for the child process to terminate using `waitpid()`.
7. Otherwise, print **"Spawn failed"** along with the error number returned in `status`.
8. Return 0 and end the program.

---

## Program

> **Note:** The parent program spawns `./child`, so the child program must be built with the executable name `child` and placed in the same directory as the parent program.

### Child Program

```c
#include <stdio.h>
#include <unistd.h>

int main(void)
{
    printf("I am the Child Process\n");
    printf("Child PID  = %d\n", (int)getpid());
    printf("Parent PID = %d\n", (int)getppid());

    return 0;
}
```

### Parent Program

```c
#include <stdio.h>
#include <spawn.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

extern char **environ;

int main(void)
{
    pid_t pid;

    char *args[] = {"child", NULL};

    printf("I am the Parent Process\n");
    printf("Parent PID = %d\n", (int)getpid());

    int status = posix_spawn(
        &pid, "./child", NULL, NULL, args, environ
    );

    if (status == 0)
    {
        printf("Child PID  = %d\n", (int)pid);
        waitpid(pid, NULL, 0);
    }
    else
    {
        printf("Spawn failed: %d\n", status);
    }

    return 0;
}
```

---

## Sample Output

```text
I am the Parent Process
Parent PID = 146
Child PID  = 147
I am the Child Process
Child PID  = 147
Parent PID = 146
```

*(PID values differ on every run. The child's Parent PID matches the parent's PID, and the Child PID printed by the parent matches the child's own PID.)*

---

## Output

<!-- Student: paste your output screenshot here, e.g. ![Output Image](output.png) -->

<br>
<br>
<br>
<br>

---

## Result
Thus, a child process was successfully created from a parent program using `posix_spawn()` on the QNX operating system running in a VMware virtual machine. The parent waited for the child to terminate using `waitpid()`, and the PIDs of the parent and child processes were displayed.

---
