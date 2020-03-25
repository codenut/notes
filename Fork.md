# Fork

A Unix system call which spawns a child process based off a parent process.
When a parent forks a new child, the child process gets a copy of the parent's file descriptors.
Ther kernel uses a descriptor reference counts to decide wheter to close a socket or not. It only closes the socket once the descriptor reference count becomes 0. The kernel increments the descriptor reference count everytime a child process gets created.

- The simplest way to write a concurrent server in Unix is to use fork() system call
- When a process forks a new process it becomes a parent process to that newly forked child process.
- Parent and child share the same file descriptors after the call to fork
- The kernel uses descriptor reference counts to decide whether to close the file/scoket or not
- The role of a server parent process: all it does now is accept a new connection from a client request, and loop over to accept a new client connection.

A zombie is a process that has terminated, but its parent has not waited for it and has not received its termination status yet.

- If you dont close duplicate descriptors, the clients won't termiante because the client connections won't get closed.
- If you dont close a duplicate descriptors, your long-running server will eventually run out of available file descriptors(max open files).
- When you fork a child process and it exits and the parent process doesn't wait for it and doesn't collect its termination status, it becomes a zomebie.
- Zombies use up memory.
- Zombies can't be killed and you need to wait for it.

Zombie process can be killed by a combination of a signal handler and a wait system call. When a child process exits, the kernel sends a SIGCHLD signal. The parent process can set up a signal handler to be asynchronously notified of that SIGCHLD event and then it can wait for the child to collect its termination status.
Use waitpid system call with WNOHANG option in a loop to take care of terminated child process.
