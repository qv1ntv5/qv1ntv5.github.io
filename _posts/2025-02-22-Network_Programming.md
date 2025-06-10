---
layout: post
title: Network Programming
subtitle: Introduction to Network programming.
tags: [C]
---
## Network Programming Introduction.

Network Programming in C involves writting applications that communicate over a network, mainly using *Sockets*.

<br>

### Understanding Socket programming.

In brief terms, **sockets** provides an interface between the application layer and the transport layer. Lets break this up.

<br>

#### The OSI Model.

In networking, the OSI model is a conceptual framework that describes how different networking protocols interact. It consists of seven layers but for us, there mainly three of it that are interesting for us:

- **Application Layer**: This is where the user, the network and the application interact between them (SSH, HTTP, etc).
- **Transport Layer**: This layer cares about how data is send between endpoints (TCP, sockets, etc).
- **Network Layer**: This layer cares about how to guide data through the network between two hosts (routing, addressing, etc).

<br>

#### Sockets.

A Socket **acts as a bridge betweem the application layer and the transport layer** allowing applications (and in extension; users) to send data over a network. THus, when an application wants to comunicate with another host, it creates a socket object which is associated with three more elements:

- An **IP Adress**; that references the host in other network.
- A **Port Number**; that identifies the receiver application protocol (HTTP, SSH, etc) and allows multiplexing network communications; several communications over one unique network.
- A **Transport Protocol**; TCP or UDP which will identify how the data is send, priorizing reliability or speed in the communications.

In summary, this socket object and his elements are thinked to allow the applications to delegate the process of sending the data through unversal *high-level programming interfaces* or *APIs* instead of dealing themselves with raw network packeting.

Thus, when the application wants to send data, make use of the socket object and calls socket's functions like *send()* or *recv()* which internally translate the data to transport layer segments, more precisely; TCP/UDP segments. Then morefurther, the transport layer encapsulates this segments over the TCP/UDP packets and send them to the network layer.

Is worth to mention that the socket transformation of the data belongs to the *transport layer* and not to the *network layer* since the socket determine how the information is sent and dont provides a way or guide to the data destination which is work of the network layer. 

<br>

### Commonly used functions and types of sockets.

There are severals methods that a socket allows:

- **socket()**: Creates a socket.
- **bind()**: Associates a socket with an IP address and port making it available for a connection.
- **listen()**: Sets up a socket to listen for incoming connections from other hosts (for TCP servers).
- **accept()**: Accepts a connection from a client.
- **connect()**: Connects a client to a server.
- **send() / recv()**: Sends and receives data over a TCP connection.
- **sendto() / recvfrom()**: Sends and receives data over a UDP connection.
- **close()**: Closes the socket.

We can learn more through the following [link](https://www.geeksforgeeks.org/socket-programming-cc/).

There are also several types of sockets:

- **Stream Sockets**: Reliable, connection-oriented communication (TCP communication).
- **Datagram Sockets**: Unreliable but quicker, connectionless communication (UDP communication).
- **Raw Sockets**: Used for low-level network operations (e.g., crafting custom packets).

<br>

### Sockets in C.

#### Defining a socket in C.

In C, define a socket requieres the *sys/socket.h* library a socket have the following structure:

```c
int sockfd = socket(domain, type, protocol);
```

This syscall returns an integer which acts like a *socket descriptor*, an integer that identifies this unboudn socket in later calls.

- The **domain**; specifies the address family used (IPv4, IPv6).
- The **type**; specifies the type of socket to be used.  
- The **protocol**; an integer that specifies the network layer protocol, 0 value is for IP. 

So for example, in a default TCP socket communication we would define the socket as follows:

```c
socket(AF_INET, SOCK_STREAM, 0);
```

Which defines an unbound socket over a IPv4 address, a TCP stream comunication and through the IP network protocol. This other one:

```c
socket(AF_INET6, SOCK_DGRAM, 0);
```

would define an unbound socket over an IPv6 address, a UDP stream communication and through a IP network protocol.

In summary, this socket() syscall returns a file descriptor that can be used in ulterior operations with sockets. If it fails, it would return a -1.

<br>

#### Socket Structure.

With the **socket()** syscall, we have defined an unbound socket, which is nothing but an empty communication channel determined by a descriptor. We now define his properties through the **sockaddr_in structure**. 

This structure have attributes that complement the socket definition above and is used in concomitance with the socket() descriptor in other syscalls like, bind() or listen().

Requires the following libraries:

```c
#include <sys/socket.h> //Internet Protocol family.
#include <netinet/in.h> //Internet address family.
#include <netinet/ip.h> //superset of previous.
```

The definition of this structure is like follows:

```c
struct sockaddr_in {
    sa_family_t    sin_family; // an integer value that defines the address family: AF_INET constant for IPv4 address
    in_port_t      sin_port;   // port in network byte order. htons(portnumber).
    struct in_addr sin_addr;   /* internet address interface: INADDR_ANY constant to allow comunications on any interfaces in the device*/
};

/* Internet address */
struct in_addr {
    uint32_t  s_addr;     /* address in network byte order */
};
```

Is worth to mention that, in the *sin_port* variable it would sound reasonable that we use directly an integer instead of bytes. The key of the question is that this data is gonna be send along the network to another hosts and some CPUs store the bytes in Big-Endian (first significative byte in the first position) and others in Little-Endian (first significative byte in the last position). 

So it is neccesary to standarize how the bytes are sent through the network and that's why we use htons. The *htons()* function order the bytes in Big-Endian. The standard to send bytes over a network is in Big-Endian. 

This structure is used with the socket descriptor with others syscalls like *listen()*, *bind()* or *connect()*.

<br>

### Socket Methods.

Now that we know how to create an empty socket and to define his properties by defining a socket structure we gonna see how to combine this two entities to transfer data through the network.

<br>

#### Bind.

Bind syscall is defined on the manual entry as follows:

```c
#include <sys/socket.h>
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

It accepts:

- **int sockfd**: Socket descriptor, given by *socket()* syscall.
- **const struct sockaddr \*addr**: The pointer to the address to bind given by the *sockaddr structure*.
- **socklen_t addrlen**: The size, in bytes, of the address structure pointed to by addr.

When a socket is created with *socket()*, it exists in a name space (address family) but has no address assigned to it. *bind()* assigns the address specified by *addr* to the socket referred to by the file descriptor *sockfd*.  *addrlen* specifies the size, in bytes, of the address structure pointed to by *addr*.  Traditionally, this operation is called “assigning a name to a socket”.

Bind would be the operation that a server performs before to start listen in an specific address and port. It would return a 0 in a successfull bind and -1 if an error ocurred.

<br>
#### Connect.

The *connect()* syscall:

```c
#include <sys/socket.h>
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

This function returns a **0** value is the connection is succeed and **-1** if doesn't succeed. It accepts the following arguments:

- **int sockfd**: Socket descriptor, given by *socket()* syscall.
- **const struct sockaddr \*addr**: The pointer to the address to bind given by the *sockaddr structure*.
- **socklen_t addrlen**: The size, in bytes, of the address structure pointed to by addr.

Thus, the *connect()* system call connects the socket referred to by the file descriptor *sockfd* to the address specified by *addr*. The *addrlen* argument specifies the size of *addr*. The format of the address in *addr* is determined by the address space of the socket *sockfd*.

<br>

#### Listen.

The listen function is defined as follows:

```c
#include <sys/socket.h>
int listen(int sockfd, int backlog);
```

The *listen()* marks the socket referred to by *sockfd* as a passive socket, that is, as a socket that will be used to accept incoming connection requests using *accept()*.

The **int backlog** is an integer that defines how many clientes can be queued waiting for them to be accepted.

<br>

#### Accept.

The accept() function is as follows:

```c
#include <sys/socket.h>
int accept(int sockfd, struct sockaddr *_Nullable restrict addr, socklen_t *_Nullable restrict addrlen);
```

The *accept()* system call is used with connection-based socket types (SOCK_STREAM, SOCK_SEQPACKET).  

It extracts the first connection request on the queue of pending connections for the listening socket, *sockfd*, creates a new connected socket, and returns a new file descriptor referring to that socket. 

The newly created socket is not in the listening state. The original socket *sockfd* is unaffected by this call.

An example of accepting connection can be:

```c
int clientSocket = accept(sockfd, NULL, NULL);
```

The *accept()* function returns the accepted socket file descriptor and -1 if the operation fails. This file operator is then used within *send()* or *recv()* functions. 
<br>

#### Send/Recv.

**Send**

The system calls send(), are used to transmit a message to another socket and it only may be used when the socket is in a *connected* state. The definition of the *send()* function is:

```c
#include <sys/socket.h>
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
```

- **int sockfd**; The file descriptor for the socket.
- **const void \*buf**; A pointer to the content buffer whcih wants to be sent.
- **size_t len**; The length of the buffer.
- **int flags**; Optional flags, 0 for standard.

On success, these calls return the number of bytes sent. On error, -1 is returned, and errno is set to indicate the error. 

An example of the send function can be:

```C
send(sockfd, msgptr, sizeof(msgptr), 0);
```

<br>

**Recv**

The *recv()*, calls are used to receive messages from a socket. They may be used to receive data on both connectionless and connection-oriented sockets. This page first describes common features of all three system calls, and then describes the differences between the calls.

The function is defined as follows:

```C
ssize_t recv(int sockfd, const void *buf, size_t len, int flags);
```
- **int sockfd**; The file descriptor for the socket.
- **const void \*buf**; A pointer to the content buffer whcih wants to be sent.
- **size_t len**; The length of the buffer.
- **int flags**; Optional flags, 0 for standard.

Pretty the same of *send()* syscall. An example can be:

```C
recv(sockfd, msgptr, sizeof(msgptr), 0);
```

<br>


## Client-Server example.

Once we have the clear the theory, we can proceed to write an example between a client that connects to a server which receives a byte of data and close connection.

The esqueme of functions and comunication is like follows:

<div style="text-align:center">
    <img src="{{ 'assets/img/C/Pasted image 20250218213858.png' | relative_url }}" text-align="center"/>
</div>


**Is worth to mention that a client can use *bind* in order to make a connection through a especific address and port but is not required**

<br>

### Server code.

Lets start with the *server* code. This is a program that creates a *socket* and binds it in an especific address, in this case in the loopback address port 9999 and waits for connection. Once it receives a connection it displays the message over the standard output.

```c
#include <netinet/in.h> //Internet Addressing library.
#include <stdio.h> //Standard input output header.
#include <stdlib.h> //Standard library for general purpouse
#include <sys/socket.h> //Socket API library.
#include <sys/types.h>  //Extended data types library.
#include <arpa/inet.h> // IPv4 address manipulation
#include <string.h>

int main(){
    
    char msgptr[255] = "Hello World!.\n"; 

    int sockfd = socket(AF_INET, SOCK_STREAM, 0); //TCP IPv4 socket.

    struct sockaddr_in sockaddr;
    sockaddr.sin_family = AF_INET;
    sockaddr.sin_port = htons(9999);
    sockaddr.sin_addr.s_addr = inet_addr("127.0.0.1"); //We only want to connet over the loopback.

    int sockbind = bind(sockfd, (struct sockaddr*)&sockaddr, sizeof(sockaddr)); 

    if (sockbind == 0){
        printf("[+] Socket binded successfully on address 127.0.0.1:9999.\n");
    }

    int socklisten = listen(sockfd, 1);

    if (socklisten == 0){
        printf("[+] Socket is listening...\n");
    }

    int clientaccept = accept(sockfd, NULL, NULL);

    if (clientaccept != -1) {
        printf("[+] Received client connection!\n");
    }

    int msgsend=send(clientaccept, msgptr, strlen(msgptr), 0);

    if (msgsend != -1){

        printf("[+] %d bytes sended to client.\n", msgsend);

    }

    

    return 0;
}
```


In the server code, step by step we:

- Create an empty TCP IPv4 socket and define the properties (family address, ip adress, port) of this socket through a structure.
- Bind the socket to the address and port defined on the structure.
- Listen in that port.
- Accept the client and pass the connection to the socket.
- Send a message and end the function.


<br>

### Client Code.


```C
#include <netinet/in.h> //Internet Addressing library.
#include <stdio.h> //Standard input output header.
#include <stdlib.h> //Standard library for general purpouse
#include <sys/socket.h> //Socket API library.
#include <sys/types.h>  //Extended data types library.
#include <arpa/inet.h> // IPv4 address manipulation
#include <string.h>

int main(){

    int sockfd = socket(AF_INET, SOCK_STREAM, 0); //TCP IPv4 socket.

    struct sockaddr_in sockaddr;
    sockaddr.sin_family = AF_INET;
    sockaddr.sin_port = htons(9999); //The port is specified to be setted in Big-Endian.
    sockaddr.sin_addr.s_addr = inet_addr("127.0.0.1"); //We define the family address family, the ip and the port we want to connect.

    int sockconnect = connect(sockfd, (struct sockaddr*)&sockaddr, sizeof(sockaddr)); //We connect to the specified address.

    if (sockconnect == 0){
        printf("[+] Connected successfully to 127.0.0.1:9999.\n"); //Check connection.
    }

	char* message = malloc(255*sizeof(char));

	int messagebytes = recv(sockfd, message, 255, 0); //receive message.

    if (messagebytes != -1){
        printf("[+] Received %d bytes\n",messagebytes);
    }

	printf("[+] The message is: %s\n",message);

	free(message); //free pointer.

    return 0;
}
```

In the client code, step by step we:

- Create an empty TCP IPv4 socket and define the properties (family address, ip adress, port) of this socket through a structure.
- Connect the socket to the address and port defined on the structure.
- Send a message and end the function.


