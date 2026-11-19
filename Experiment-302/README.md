# Experiment-302 : Message Passing with Multiple Request Types between Client and Server in QNX

---

## Objective  
To create, build, and execute a menu-driven client program and a server program in QNX that communicate using message passing. The client sends a request containing a request type and two integers. The server uses a `switch` statement on the request type to perform addition, multiplication, or return the current date and time, and sends the reply back to the client. The request and reply structures are shared through a common header file (`common.h`). The programs run on QNX inside a VMware virtual machine.

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

### Server Program

1. Start the server program.
2. Include `common.h`, which defines the request types `MSG_ADD`, `MSG_MULTIPLY` and `MSG_GET_TIME`, and the structures `request_t` and `reply_t`.
3. Create a channel using `ChannelCreate()`. If it fails, print an error message and terminate.
4. Print **"Message Passing Server Started"**, the server PID and the channel ID (CHID).
5. Repeat the following steps continuously:
   1. Print **"Waiting for client request..."**.
   2. Wait for a request from a client using `MsgReceive()`. If it fails, print an error message and go back to step 5.1.
   3. Clear the reply structure using `memset()`.
   4. Print the request type received.
   5. Use a `switch` statement on `req.type`:
      * **MSG_ADD:** store `req.a + req.b` in `reply.result` and print the addition.
      * **MSG_MULTIPLY:** store `req.a * req.b` in `reply.result` and print the multiplication.
      * **MSG_GET_TIME:** get the current time using `time()`, convert it using `localtime_r()`, and format it as `dd-mm-yyyy hh:mm:ss` in `reply.time_str` using `strftime()`. If the conversion fails, send an error using `MsgError()` and go back to step 5.1.
      * **default:** print **"Invalid request type"**, send an error using `MsgError()` and go back to step 5.1.
   6. Send the reply to the client using `MsgReply()` and print **"Reply sent successfully"**.

### Client Program

1. Start the client program.
2. Check that exactly two command-line arguments (server PID and CHID) are given. Otherwise, print the usage message and terminate.
3. Convert the PID and CHID to integers using `atoi()`.
4. Connect to the server's channel using `ConnectAttach()`. If it fails, print an error message and terminate.
5. Print **"Connected to QNX server"**.
6. Repeat the following steps continuously:
   1. Display the menu: 1. Addition, 2. Multiplication, 3. Get Current Date and Time, 4. Exit.
   2. Read the user's choice. If the input is not a valid number, print an error message, disconnect and terminate.
   3. Use a `switch` statement on the choice:
      * **1:** set the request type to `MSG_ADD` and read two numbers from the user.
      * **2:** set the request type to `MSG_MULTIPLY` and read two numbers from the user.
      * **3:** set the request type to `MSG_GET_TIME`.
      * **4:** print **"Client exiting..."**, disconnect using `ConnectDetach()` and terminate.
      * **Any other value:** print **"Invalid choice. Try again."** and go back to step 6.1.
   4. Send the request to the server using `MsgSend()` and wait for the reply. If it fails, print an error message and go back to step 6.1.
   5. If the choice was 1 or 2, display the result received from the server. If the choice was 3, display the date and time received from the server.

---

## Program

> **Note:** Keep `common.h`, the server program and the client program in the same directory. Run the server program first. It prints its PID and CHID. Then run the client program on the same QNX system as `./client <PID> <CHID>`.

### common.h

```c
#ifndef COMMON_H
#define COMMON_H

#define MSG_ADD       1
#define MSG_MULTIPLY  2
#define MSG_GET_TIME  3

typedef struct
{
    int type;
    int a;
    int b;
} request_t;

typedef struct
{
    int result;
    char time_str[64];
} reply_t;

#endif
```

### Server Program

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <string.h>
#include <errno.h>
#include <sys/neutrino.h>

#include "common.h"

int main(void)
{
    int chid;
    int rcvid;

    request_t req;
    reply_t reply;

    /* Create communication channel */
    chid = ChannelCreate(0);

    if (chid == -1)
    {
        perror("ChannelCreate failed");
        return 1;
    }

    printf("Message Passing Server Started\n");
    printf("Server PID = %d\n", getpid());
    printf("CHID = %d\n", chid);

    /* Continuously wait for client requests */
    while (1)
    {
        printf("\nWaiting for client request...\n");

        rcvid = MsgReceive(chid, &req,
                           sizeof(req), NULL);

        if (rcvid == -1)
        {
            perror("MsgReceive failed");
            continue;
        }

        memset(&reply, 0, sizeof(reply));

        printf("Request received. Type = %d\n",
               req.type);

        switch (req.type)
        {
            case MSG_ADD:

                reply.result = req.a + req.b;

                printf("Addition: %d + %d = %d\n",
                       req.a, req.b, reply.result);
                break;

            case MSG_MULTIPLY:

                reply.result = req.a * req.b;

                printf("Multiplication: %d * %d = %d\n",
                       req.a, req.b, reply.result);
                break;

            case MSG_GET_TIME:
            {
                time_t now;
                struct tm time_info;

                now = time(NULL);

                if (localtime_r(&now, &time_info) == NULL)
                {
                    MsgError(rcvid, EINVAL);
                    continue;
                }

                strftime(reply.time_str,
                         sizeof(reply.time_str),
                         "%d-%m-%Y %H:%M:%S",
                         &time_info);

                printf("Current time: %s\n",
                       reply.time_str);

                break;
            }

            default:

                printf("Invalid request type\n");

                MsgError(rcvid, ENOSYS);
                continue;
        }

        /* Send reply to client */
        if (MsgReply(rcvid, EOK,
                     &reply, sizeof(reply)) == -1)
        {
            perror("MsgReply failed");
        }
        else
        {
            printf("Reply sent successfully\n");
        }
    }

    ChannelDestroy(chid);

    return 0;
}
```

### Client Program

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/neutrino.h>
#include <sys/netmgr.h>

#include "common.h"

int main(int argc, char *argv[])
{
    int pid;
    int chid;
    int coid;
    int choice;

    request_t req;
    reply_t reply;

    /* Check command-line arguments */
    if (argc != 3)
    {
        printf("Usage: ./client <PID> <CHID>\n");
        return 1;
    }

    pid = atoi(argv[1]);
    chid = atoi(argv[2]);

    /* Connect to server */
    coid = ConnectAttach(
        ND_LOCAL_NODE,
        pid,
        chid,
        _NTO_SIDE_CHANNEL,
        0
    );

    if (coid == -1)
    {
        perror("ConnectAttach failed");
        return 1;
    }

    printf("Connected to QNX server\n");

    while (1)
    {
        printf("\n===== MESSAGE PASSING CLIENT =====\n");
        printf("1. Addition\n");
        printf("2. Multiplication\n");
        printf("3. Get Current Date and Time\n");
        printf("4. Exit\n");

        printf("Enter your choice: ");
        fflush(stdout);

        if (scanf("%d", &choice) != 1)
        {
            printf("Invalid input\n");
            ConnectDetach(coid);
            return 1;
        }

        req.a = 0;
        req.b = 0;
        req.type = choice;

        switch (choice)
        {
            case 1:

                req.type = MSG_ADD;

                printf("Enter first number: ");
                fflush(stdout);

                if (scanf("%d", &req.a) != 1)
                {
                    printf("Invalid input\n");
                    ConnectDetach(coid);
                    return 1;
                }

                printf("Enter second number: ");
                fflush(stdout);

                if (scanf("%d", &req.b) != 1)
                {
                    printf("Invalid input\n");
                    ConnectDetach(coid);
                    return 1;
                }

                break;

            case 2:

                req.type = MSG_MULTIPLY;

                printf("Enter first number: ");
                fflush(stdout);

                if (scanf("%d", &req.a) != 1)
                {
                    printf("Invalid input\n");
                    ConnectDetach(coid);
                    return 1;
                }

                printf("Enter second number: ");
                fflush(stdout);

                if (scanf("%d", &req.b) != 1)
                {
                    printf("Invalid input\n");
                    ConnectDetach(coid);
                    return 1;
                }

                break;

            case 3:

                req.type = MSG_GET_TIME;
                break;

            case 4:

                printf("Client exiting...\n");
                ConnectDetach(coid);
                return 0;

            default:

                printf("Invalid choice. Try again.\n");
                continue;
        }

        /* Send request and receive reply */
        if (MsgSend(
                coid,
                &req,
                sizeof(req),
                &reply,
                sizeof(reply)
            ) == -1)
        {
            perror("MsgSend failed");
            continue;
        }

        /* Display server response */
        if (choice == 1 || choice == 2)
        {
            printf("\nResult received from server = %d\n",
                   reply.result);
        }
        else if (choice == 3)
        {
            printf("\nCurrent date and time = %s\n",
                   reply.time_str);
        }
    }

    ConnectDetach(coid);

    return 0;
}
```

---

## Sample Output

### Server Terminal

```text
Message Passing Server Started
Server PID = 4098
CHID = 1

Waiting for client request...
Request received. Type = 1
Addition: 12 + 8 = 20
Reply sent successfully

Waiting for client request...
Request received. Type = 2
Multiplication: 6 * 7 = 42
Reply sent successfully

Waiting for client request...
Request received. Type = 3
Current time: 19-11-2026 15:45:30
Reply sent successfully

Waiting for client request...
```

### Client Terminal

```text
Connected to QNX server

===== MESSAGE PASSING CLIENT =====
1. Addition
2. Multiplication
3. Get Current Date and Time
4. Exit
Enter your choice: 1
Enter first number: 12
Enter second number: 8

Result received from server = 20

===== MESSAGE PASSING CLIENT =====
1. Addition
2. Multiplication
3. Get Current Date and Time
4. Exit
Enter your choice: 2
Enter first number: 6
Enter second number: 7

Result received from server = 42

===== MESSAGE PASSING CLIENT =====
1. Addition
2. Multiplication
3. Get Current Date and Time
4. Exit
Enter your choice: 3

Current date and time = 19-11-2026 15:45:30

===== MESSAGE PASSING CLIENT =====
1. Addition
2. Multiplication
3. Get Current Date and Time
4. Exit
Enter your choice: 4
Client exiting...
```

*(The server PID, CHID and the date and time differ on every run. The server keeps running and waits for the next client after the client exits.)*

---

## Output

<!-- Student: paste your output screenshots here (server terminal and client terminal), e.g. ![Output Image](output.png) -->

<br>
<br>
<br>
<br>

---

## Result
Thus, a menu-driven message passing client and a server were successfully implemented on the QNX operating system running in a VMware virtual machine. The client sent requests of different types (addition, multiplication and current date and time), and the server processed each request using a `switch` statement and sent the reply, which the client displayed.

---
