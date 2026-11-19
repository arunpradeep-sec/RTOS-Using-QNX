# Experiment-202 : Thread Synchronization using Mutex in QNX: VMware Virtual Machine

---

## Objective  
To create, build, and execute a multithreaded C program in QNX in which two threads increment a shared counter, using a POSIX mutex (`pthread_mutex_t`) to protect the critical section and prevent a race condition, on QNX running inside a VMware virtual machine.

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
2. Declare the shared global variable `counter` and initialize it to 0.
3. Initialize the mutex `data_mutex` using `PTHREAD_MUTEX_INITIALIZER`.
4. In `main()`, create two threads, **"Thread 1"** and **"Thread 2"**, using `pthread_create()`. Both threads execute the function `increment_counter()` and receive their name as the argument.
5. In `increment_counter()`, repeat the following 5 times:
   * Lock the mutex using `pthread_mutex_lock()`.
   * **Critical section:** increment `counter` and print the thread name and the current counter value.
   * Unlock the mutex using `pthread_mutex_unlock()`.
6. Return from the thread function.
7. In `main()`, wait for both threads to finish using `pthread_join()`.
8. Print the final counter value.
9. Return 0 and end the program.

---

## Program

```c
#include <stdio.h>
#include <pthread.h>

int counter = 0;

/* Initialize the mutex */
pthread_mutex_t data_mutex = PTHREAD_MUTEX_INITIALIZER;

/* Function executed by each thread */
void *increment_counter(void *arg)
{
    char *thread_name = (char *)arg;

    for (int i = 0; i < 5; i++)
    {
        /* Lock the mutex */
        pthread_mutex_lock(&data_mutex);

        /* Critical section */
        counter++;
        printf("%s: Counter = %d\n", thread_name, counter);

        /* Unlock the mutex */
        pthread_mutex_unlock(&data_mutex);
    }

    return NULL;
}

int main(void)
{
    pthread_t thread1, thread2;

    /* Create two threads */
    pthread_create(&thread1, NULL, increment_counter, "Thread 1");
    pthread_create(&thread2, NULL, increment_counter, "Thread 2");

    /* Wait for both threads to finish */
    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    printf("\nFinal Counter Value = %d\n", counter);

    return 0;
}
```

---

## Sample Output

```text
Thread 2: Counter = 1
Thread 2: Counter = 2
Thread 2: Counter = 3
Thread 2: Counter = 4
Thread 2: Counter = 5
Thread 1: Counter = 6
Thread 1: Counter = 7
Thread 1: Counter = 8
Thread 1: Counter = 9
Thread 1: Counter = 10

Final Counter Value = 10
```

*(Which thread runs first, and how the two threads take turns, may differ from run to run. Because the mutex protects the counter, the values are always printed in order from 1 to 10 and the final value is always 10.)*

---

## Output

<!-- Student: paste your output screenshot here, e.g. ![Output Image](output.png) -->

<br>
<br>
<br>
<br>

---

## Result
Thus, a multithreaded program using a POSIX mutex was successfully developed, compiled, and executed on the QNX operating system running in a VMware virtual machine. The two threads updated the shared counter safely, and the final counter value was 10.

---
