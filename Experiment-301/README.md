# Experiment-301 : Message Passing between Client and Server in QNX

---

## Objective  
To create, build, and execute a server program and a client program in QNX that communicate using synchronous message passing. The client sends two integers to the server using `MsgSend()`, the server receives them using `MsgReceive()`, adds them, and returns the sum using `MsgReply()`, on QNX running inside a VMware virtual machine.

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
2. Define the request structure `request_t` containing two integers, `a` and `b`.
3. Create a channel using `ChannelCreate()` and store the channel ID (`chid`).
4. Print the server PID using `getpid()` and the channel ID (CHID).
5. Print **"Waiting for client..."**.
6. Wait for a message from the client using `MsgReceive()`. The program blocks here until a message arrives. The request is stored in `req` and the receive ID is stored in `rcvid`.
7. Calculate the result: `result = req.a + req.b`.
8. Print the two received numbers.
9. Send the result back to the client using `MsgReply()` with the receive ID `rcvid`.
10. Print the reply that was sent.
11. Return 0 and end the program.

### Client Program

1. Start the client program.
2. Read the server PID from the first command-line argument and the CHID from the second command-line argument, and convert both to integers using `atoi()`.
3. Connect to the server's channel using `ConnectAttach()` and store the connection ID (`coid`).
4. Prepare the request with `a = 12` and `b = 8`.
5. Send the request to the server using `MsgSend()`. The client blocks until the server replies, and the reply is stored in `reply`.
6. Print the result received from the server.
7. Disconnect from the channel using `ConnectDetach()`.
8. Return 0 and end the program.

---

## Program

> **Note:** Run the server program first. It prints its PID and CHID. Then run the client program on the same QNX system, giving the server PID as the first argument and the CHID as the second argument.

### Server Program

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/neutrino.h>
#include <errno.h>

typedef struct {
    int a;
    int b;
} request_t;

int main(void)
{
    int chid = ChannelCreate(0);

    printf("Server PID = %d\n", getpid());
    printf("CHID = %d\n", chid);

    request_t req;
    int result;

    printf("Waiting for client...\n");

    int rcvid = MsgReceive(chid, &req, sizeof(req), NULL);

    result = req.a + req.b;

    printf("Received: %d and %d\n", req.a, req.b);

    MsgReply(rcvid, EOK, &result, sizeof(result));

    printf("Reply sent: %d\n", result);

    return 0;
}
```

### Client Program

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/neutrino.h>
#include <sys/netmgr.h>

typedef struct {
    int a;
    int b;
} request_t;

int main(int argc, char *argv[])
{
    int pid = atoi(argv[1]);
    int chid = atoi(argv[2]);

    int coid = ConnectAttach(
        ND_LOCAL_NODE, pid, chid,
        _NTO_SIDE_CHANNEL, 0);

    request_t req = {12, 8};
    int reply;

    MsgSend(coid, &req, sizeof(req),
            &reply, sizeof(reply));

    printf("Result received = %d\n", reply);

    ConnectDetach(coid);

    return 0;
}
```

---

## Sample Output

### Server Terminal

```text
Server PID = 4098
CHID = 1
Waiting for client...
Received: 12 and 8
Reply sent: 20
```

### Client Terminal

```text
Result received = 20
```

*(The server PID and CHID values differ on every run. These are the values that are passed to the client as command-line arguments.)*

---

## Output

<!-- Student: paste your output screenshot here, e.g. ![Output Image](output.png) -->

<br>
<br>
<br>
<br>

---

## Result
Thus, synchronous message passing between a client and a server was successfully implemented on the QNX operating system running in a VMware virtual machine. The client sent the numbers 12 and 8 to the server using `MsgSend()`, the server returned their sum using `MsgReply()`, and the client displayed the result 20.

---
