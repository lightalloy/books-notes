Processes: any code is executed in a process.
Pid - process id.
Ppid - process parent id.

File descriptors - represent open files(resources). Devices, sockets, pids and pipes  (resources) are also treated like files.
`file.fileno` - unique number of the open resource.
Standart streams: STDIN (input), STDOUT (output), STDERR (error), they are also resources with fileno 0, 1, 2.
Processes have limits for the number of file descriptors. The number is usually very large. You can change it.

*Environment variables* are often used to accept input to a program. They are global to a process and are inhereted by child processes.

`$PROGRAM_NAME`, `$0` - process names

Exit codes - last mark of a program, when it exits.
0 - success, 1 - unsuccessful (usually), you can use custom codes.
`at_exit` hook - executed at exit
`abort` - exit unsuccessfully (code 1), executes `at_exit` before ending

*Forking* - creating a copy of an existing process. Copies all the memory and file descriptors of the parent process.
`fork` may be used with `if` or more often with a block.
`fork` returns 2 times - for the child and the parent processes.

Copy-on-write - copying only when there's a need to modify, available for forking since ruby 2.0

*Orphaned processes* - child processes that run after the parent dies. They're not treated differently from other processes.

`Process.wait` - a blocking call, the parent process will wait for the child to exit. Returns status of an exited child.
`wait2` - return child pid and status
`waitpid`, `waitpid2` - wait for the specific process.
Waiting when no children are running will cause `Errno::ECHILD`

*Zombie Processes* - dead processes, whose status hasn't been waited on.
If you want to "fire and forget" you need to detauch process.

Nonblocking calls to keep track of children:
trapping the :CHLD signal - `trap(:CHLD)` (will tell the parent when the child exits)
`Process.wait(-1, Process::WNOHANG)` - nonblocking call

*Signals*
Signals are sent from one process to another process, using the kernel as a middleman.
You can send signals like `SIGINT`, `SIGKILL` or others to a process (`SIG` part is optional). Signals have their default actions like terminate, terminate and dump, ignore, pause or resume.
`SIGUSR1`, `SIGUSR2` - special user-defined signals for the process.

You can redefine signals, e.g. `trap(:INT) { print "Na na na, you can't get me" }` `SIGKILL` can't be redefined.

Ignoring signals - `trap(:INT, "IGNORE")`

*IPC* - inter-process communication
Can be done with pipes or sockets.
*Pipes* - one-way communication. One end is a reader, another is a writer.
*Sockets* - 2-way communication, reading and writing.
`child_socket, parent_socket = Socket.pair(:UNIX, :DGRAM, 0)`

*Remote IPC* - RPC, ZeroMQ, TCP sockets.

*Daemon processes* - processes that run in the background, not under the control of a user or at a terminal. `Process.daemon`

*Process group* - usually a group is a parent with children, but you can also set a group arbitrarily.
*Session Groups* - `git log | grep shipped | less` - each command will have it's own process group, but the same session group.

`exec` - transform ruby process to any other process (pass string or array), won't go back
`system` - returns true or false
``` and `%x[]` - returns output of a program
`Process#spawn` - nonblocking call
`IO.popen`, `open3` - sets up a pipe to communicate with the spawned process









