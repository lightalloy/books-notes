# Working with Unix Processes

manpages sections

• Section 1: General Commands
• Section 2: System Calls
• Section 3: C Library Functions
• Section 4: Special Files

## Chapter 3. Processes: The Atoms of Unix
*any code that is executed happens inside a process*
One process can spawn and manage many others.

### Processes Have IDs
Every process running on your system has a unique process identifier, hereby referred to as 'pid'.

```ruby
puts Process.pid
```

### Cross Referencing
```bash
ps -p <pid-of-the-process>
```

### In the Real World
Process id can often be written into logs.
You can also find the information about the process by the commands like `top` or `lsof`.

### System Calls
`$$` - global variable that holds the value of the current PID.
But `Process.pid` is much more expressive.

## Chapter 4. Processes Have Parents
```ruby
puts Process.ppid
```
Parent for the `irb` process will be a bash process.
There's not much real use of the `ppid`. It may be useful for detecting daemon processes.

## Chapter 5. Processes Have File Descriptors
Pids represent running processes, and file descriptors represent open files.

### Everything is a file
Devices, sockets, pids and pipes are treated like files.
You can use the word 'resource' when talking about files in the general sense.

### Descriptors Represent Resources
Any time that you open a resource in a running process it is assigned a file descriptor number.
File descriptors are not shared between processes, they live and die inside a process.
In Ruby, open resources are represented by the IO class.
```ruby
passwd = File.open('/etc/passwd')
puts passwd.fileno
```
Any resource is given a unique number, the kernel keeps track of the resources with the help of these numbers.

- File descriptors are assigned the lowest unused number
- once a resource is closed its file descriptor number becomes available again.

Closed resources are not given a file descriptor number.
An attempt to access it will cause an exception.
```ruby
passwd = File.open('/etc/passwd')
puts passwd.fileno
passwd.close
puts passwd.fileno
# => closed stream (IOError)
```

### Standard Streams
Every Unix process comes with three open resources:
- STDIN - standart input
- STDOUT - standart output
- STDERR - standart error

```ruby
puts STDIN.fileno
puts STDOUT.fileno
puts STDERR.fileno
```

Many methods on Ruby's IO class map to system calls of the same name.
E.g. open(2), close(2), read(2), write(2), pipe(2), fsync(2), stat(2).

## Chapter 6. Processes Have Resource Limits
There are some resource limits imposed on a process by the kernel.
The maximum number of allowed file descriptors:
```ruby
p Process.getrlimit(:NOFILE)
# => [1024, 1048576]
```
### Soft Limits vs. Hard Limits
First element -- soft limit.
If it will be exceeded, the exception will be raised.
The hard limit is a large number, you'll probably won't be able to open so much resources because of the hardware constraints.

### Bumping the Soft Limit
You can change the soft limit:
```ruby
Process.setrlimit(:NOFILE, 4096)
# will set both soft and hard limits to 4096
```

Exceeding the soft limit will raise `Errno::EMFILE`

### Other Resources
Check and modify limits of other system resources:
```ruby
# The maximum number of simultaneous processes
# allowed for the current user.
Process.getrlimit(:NPROC)
# The largest size file that may be created.
Process.getrlimit(:FSIZE)
# The maximum size of the stack segment of the
# process.
Process.getrlimit(:STACK)
```

### In the Real World
You won't need to change our process limits often.
One of the cases is: dealing with a process which needs to handle thouthands of network connections, e.g. to test performance.
You can also want to change limits when dealing with some third-party code.

### System Calls
`Process.getrlimit` `Process.setrlimit`

## Chapter 7. Processes Have an Environment
Environment variables are key-value pairs that hold data for a process.
Processes inherit env variables from their parents.
Environment variables are per-process and are global to each process.

```bash
$ MESSAGE='wing it' ruby -e "puts ENV['MESSAGE']"
```
Equal to:
```ruby
ENV['MESSAGE'] = 'wing it'
system "echo $MESSAGE"
```

### Hash like
`ENV` uses hash-like API but is not a hash.
It implements `Enumerable` and some of the hash API, but not all `Hash` methods are implemented, so don't rely on it.

```ruby
puts ENV['EDITOR']
puts ENV.has_key?('PATH')
puts ENV.is_a?(Hash)
```

### In the Real World
```bash
RAILS_ENV= production rails server
```

Environment variables are often used as a generic way to accept input into a command-line program. Using environment variables is often less overhead than explicitly parsing command line options.

### System calls
C-library functions like `setenv(3)`, `getenv(3)`, also `environ(3)`

## Chapter 8. Processes Have Arguments
Every process has access to a special array called `ARGV`.
argv - Argument Vector.
```bash
$ cat argv.rb
p ARGV
$ ruby argv.rb foo bar -va
["foo", "bar", "-va"]
```
`argv` is simply an `Array`.

### In the Real World
Mostly accepting filenames or parsing command line command line input.
You can use libraries like `optparse` or parse them by hand.

## Chapter 9. Processes Have Names
Unix processes have very few inherent ways of communicating about their state.
- Logfiles, but they operate on the filesystem level and are not inherent to the processes themselves
- Using network (opening sockets) to communicate with other processes, but it also operates on the different level
On the process level such mechanisms are: process names and exit codes.

### Naming Processes
Every process has its name, e.g. `irb`
You can access process name with the `$PROGRAM_NAME` variable.
You can change it and so, change the name of the current process.

In ruby you can only change `$PROGRAM_NAME` or its alias `$0`, no other ways to change the process name.

## Chapter 10. Processes Have Exit Codes
When a process ends it has a last chance to make a mark: it's exit code (0 - 255)
0 is usually for succesful exits.

### How to Exit a Process
#### exit
```ruby
# hook that will be executed before exit
at_exit { puts 'Last!' }
# setting a custom exit code
exit 4
```
#### exit!
Sets an unsuccessful status code by default (1).
Won't run `at_exit` hook.
```ruby
# ypu can specify a custom code
exit! 3
```

#### abort
Kernel#abort provides a generic way to exit a process unsuccessfully.
It'll set the exit code to 1.

`at_exit` block will be invoked
```ruby
# you can set a message which will be printed to STDERR
abort "Something went horribly wrong."
```

#### raise
If an unhandled exception will be raised, it'll cause the process to end.
`Kernel#raise` won't end the process immediately, cause an exception may be rescued later.

## Chapter 11. Processes can fork
`fork(2)` - creating a copy of the existing process.
The new process will be a child of the current one.
It'll have a copy of all the memory used by and file descriptors belonging to the parent process.
In this way the two processes can share open files, sockets, etc.

Can be used with `if` (not very common):
```ruby
puts "parent process pid is #{Process.pid}"
if fork
  puts "entered the if block from #{Process.pid}"
else
  puts "entered the else block from #{Process.pid}"
end
# parent process is 21268
# entered the if block from 21268
# entered the else block from 21282
```
The if block is being executed by the parent process, while the code in the else block is being executed by the child process. The child process will exit after executing the else branch, the parent process will continue to run.

`puts fork` returns 2 different values. First is a child process pid (returned from parent), second - `nil` from the child process

### Multicore Programming?
Your code is able, but not guaranteed, to be distributed across multiple CPU cores.
Forking may lead to a `fork bomb`, cause it will need a n*parent_process_memory memory.

### Using a Block
`fork` is more commonly used with a block in ruby:
```ruby
fork { puts Process.pid }
```

### System Calls
`Kernel#fork` maps to `fork(2)`

## Chapter 12. Orphaned Processes
### Out of Control
When this code is executed, a parent will die before the child process is finished.
The child process becomes orphaned, but it'll continue doing its job (printing into STDOUT).
```ruby
fork do
  5.times do
    sleep 1
    puts "I'm an orphan!"
  end
end
abort "Parent process died..."
```

### Abandoned Children
The OS doesn't treat children differently then other processes.

### Managing Orphans
- daemon processing
- managing processes that're not attached to the terminal (see in later chapters)

## Chapter 13. Processes Are Friendly
### Being CoW Friendly
CoW delays the actual copying of memory until it needs to be written.
CoW-friendly forking: on `fork` process memory is not copied. It's only copied when that data needs to be modified.

```ruby
arr = [1, 2, 3]
fork { p arr } # no copying
fork { arr.push 4 } # copying cause of the need to write
```
Before the version 2.0, ruby used Mark-and-Sweep garbage collector, which needed to modify data while doing it's "mark" step, so the CoW wouldn't be effective.
Since ruby 2.0 it change its GC method, so now forking is CoW-friendly.

## Chapter 14. Processes Can Wait
The technique when the parent process exits before the child one is only suitable for the "fire and forget" situation.
It's useful to do smth asynchronously in a child process.

### Babysitting
But sometimes we need to keep track on the child processes.
`Process.wait` is a blocking call instructing the parent process to wait for one of its child processes to exit before continuing.

```ruby
fork { 5.times { sleep 1; p 'hi'; } }
Process.wait
abort 'end1'
```

### Process.wait and Cousins
Sometimes you have several children and want to know which one exited.
Then you can use the `Process.wait` return value (pid)

### Communicating with Process.wait2
`Process.wait2` returns `pid` and `status`.
The `status` returned from `Process.wait2` is an instance of `Process::Status`, it has a lot of useful information attached to it.

### Waiting for Specific Children
You can use `Process.waitpid` and `Process.waitpid2` to wait for the specific children.

Actually `Process.wait` and `Process.waitpid` are aliases. So are `Process.wait2` and `Process.waitpid2`
You can pass pid to `wait`, `wait2` and you can pass `-1` to `waitpid`, `waitpid2` to say "no pid".

But you should use these methods as specified before to show clear intentions.

### Race Conditions
If a child process exits before the `Process.wait` is executed, nothing happens.
So the code is free of race conditions.
But calling `Process.wait`, when there are no child processes, will raise `Errno::ECHILD`. It's always a good idea to keep track of how many child processes you have created.

### Real World
The idea of looking in on your child processes is at the core of a common Unix programming pattern. It's sometimes called babysitting processes, master/
worker, or preforking.

The core: a parent process forks several child processes and keeps track of them.
E.g. `Unicorn` uses this pattern.

### System Calls
`Process.wait` => `waitpid(2)`

## Chapter 15. Zombie Processes
`Process.wait` can be called even the long time after the process finished.
So the Kernel will retain the status of exited child processes until the parent process will call `Process.wait`.
So if you don't `wait`, the kernel will keep that information forever. It's not a good use of the resources.
If you want to fork processes in "fire and forget" manner, you need to detouch them.

```ruby
pid = fork { sleep 1 }
puts pid
sleep
```

```bash
ps -ho pid,state -p [pid of a zombie process]
# => Z
```
Any dead process whose status hasn't been waited on is a zombie process.

gem `spawnling` - makes sure that the processes are properly detauched.

## Chapter 16. Processes Can Get Signals
`Process.wait` is a nice way to keep track of children, but it's a blocking call.
What if the parent wants to do it's own job while keeping track of children?

### Trapping SIGCHILD
```ruby
...
# By trapping the :CHLD signal our process will be notified by the kernel
# when one of its children exits.
trap(:CHLD) do
  # Since Process.wait queues up any data that it has for us we can ask for it
  # here, since we know that one of our child processes has exited.
  puts Process.wait
  dead_processes += 1
  # We exit explicitly once all the child processes are accounted for.
  exit if dead_processes == child_processes
end
# ... parent process does some job
```
### SIGCHILD and Concurrency
Signal delivery is unreliable. If your code is handling a CHLD signal while another child process dies you may or may not receive a second CHLD signal.

To properly handle CHLD you must call Process.wait in a loop and look for as many dead child processes as are available.
`Process.wait` is a blocking call, but you can pass a second argument to it.
`Process.wait(-1, Process::WNOHANG)` - nonblocking.

```ruby
child_processes = 3
dead_processes = 0

#... forking

$stdout.sync = true
trap(:CHLD) do
  # Since Process.wait queues up any data that it has for us we can ask for it
  # here, since we know that one of our child processes has exited.
  # We loop over a non-blocking Process.wait to ensure that any dead child
  # processes are accounted for.
  begin
    while pid = Process.wait(-1, Process::WNOHANG)
      puts pid
      dead_processes += 1
      # We exit ourselves once all the child processes are accounted for.
      puts 'exiting'; exit if dead_processes == child_processes
    end
  rescue Errno::ECHILD
    puts 'rescued'
  end
end
#... parent job
```
### Signals Primer
Signals are asynchronous communication.
When a process receive a signal from the kernel it can: ignore the signal, perform a specified action, perform the default action.

### Where do Signals Come From?
Signals are sent from one process to another process, using the kernel as a middleman.
The original purpose of signals was to specify different ways that a process should be killed.
Killing one process from another: `Process.kill(:INT, <pid of the process>)`

### The Big Picture
Default actions of the signals:
`Term` - terminate
`Core` - terminate and dump core
`Ign` - ignore the signal
`Stop` - pause
`Cont` - resume
See the table in the book for the whole picture.
Signal names start with `SIG` (`SIGINT`), but this part is optional.

By default most signals will terminate a process.
`SIGUSR1` and `SIGUSR2` signals - defined specifically for your process.

### Redefining Signals
`trap(:INT) { print "Na na na, you can't get me" }`
=> you won't be able to interrupt a process from the code or with `Ctrl + C`

### Ingoring
`trap(:INT, "IGNORE")`

### Signal Handlers are Global
Trapping a signal is a bit like using a global variable, it's only suitable in certain situations.

### Being Nice about Redefining Signals
There is no way to preserve system default behaviour, but you can preserve other Ruby handlers that have been defined.

Preserve any previous QUIT handlers that have been defined.
```ruby
old_handler = trap(:QUIT) {
  # do some cleanup
  puts 'All done!'
  old_handler.call if old_handler.respond_to?(:call)
}
```
Process can receive signals at any given time.

### In the Real World
With signals, any process can communicate with any other process on the system, if it knows its pid.
Signals are mostly used by long running processes like servers and daemons.
Unicorn -- see `SIGNALS` file.

## Chapter 17. Processes Can Communicate
IPC - inter-process communication
2 most common ways: pipes and socket pairs

### Pipes
A pipe is a unidirectional stream of data. One process is its "end" and the other is the second "end".
One process is the reader, another is the writer, not vice versa.

```ruby
reader, writer = IO.pipe # [IO, IO]
writer.write("Into the pipe I go...")
writer.close
puts reader.read
```
`IO` is a superclass to `File`, `TCPSocket`, `UDPSocket` and others.
If you skip closing the writer then the reader will <b></b>lock and continue to read indefinitely.

### Sharing Pipes
You can use a pipe to communicate between child and parent processes.

### Streams vs. Messages
In IO there's no beginning or end.
When reading data from that IO stream you read it in one chunk at a time, stopping when you come across the delimiter

### Sockets:
```ruby
require 'socket'
child_socket, parent_socket = Socket.pair(:UNIX, :DGRAM, 0) #=> [#<Socket:fd 15>, #<Socket:fd 16>]
```
`process.recv` => read
`process.send` => write

### Remote IPC?
IPC - communication only between processes on the same machine.
Remote IPC:
- TCP sockets
- RPC
- ZeroMQ

## Chapter 18. Daemon Processes
Daemon processes - processes that run in the background, not under the control of a user or at a terminal.
Many processes are running in the background when the os is running.

### The First Process
When the kernel is bootstrapped it spawns a process called the `init` process.
It has `ppid` 0 and `pid` 1. It's the grandparent of all processes.

### Creating a Daemon Process
Example - `rackup` command, it has an option to daemonize the process.
In ruby `1.9+` there is a command `Process.daemon`, but what it does exactly?

```ruby
# exit the parent process, keep the child running
exit if fork
# The process becomes a session leader of a new session
# The process becomes the process group leader of a new process group
# The process has no controlling terminal
# returns the id of the new session group it create
Process.setsid
# The forked process that had just become a process group and session group leader forks again and then exits,
exit if fork
# changes the current working directory to the root directory
# avoids problems where the directory that the daemon was started from gets deleted or unmounted
Dir.chdir "/"
# sets all the standart streams to /dev/null
# daemon doesn't use them
# but some programs expect them to always be available
STDIN.reopen "/dev/null"
STDOUT.reopen "/dev/null", "a"
STDERR.reopen "/dev/null", "a"
```

When a process is orphaned it's `ppid` will be 1.

### Process Groups and Session Groups
Refer to how processes are handled by the terminal.
Each process belongs to a group, and each group has a unique integer id.
Usually a group is a parent with children, but you can also set a group arbitrarily:
```ruby
Process.setpgrp(new_group_id)
```
If you fork a process, the group id will be the parent process pid.
When the parent process is being controlled by a terminal and is killed by a signal the child processes will be also killed.

#### Session Groups
A session group is one level of abstraction higher up then the process group.
E.g. `git log | grep shipped | less` - each command will have it's own process group, but the same session group.
Ctrl-C will kill them all.

## Chapter 19. Spawning Terminal Processes

### fork + exec
`exec` allows us to transform our ruby process to any other process
E.g. `ls` or a python script  
`exec 'ls', '--help'`
Your ruby process will be transformed and won't go back.  
To fix that you can use `fork` to create a new process and `exec` to transform it to another process.
If you need the ouput of the `exec` process, you can use `Process.wait` to wait for it.

### Arguments to exec
Pass string => start up a shell process and the process'll interpret the shell
Pass array => skip the shell and set up the array directly as the `ARGV` to the new process.

Avoid passing a string where possible.

#### Kernel#system
`system('ls', '--help')`  
return value: `true` if the exit code is `0`, `false` if there is another code.
`nil` if execution failed

#### Kernel#`
``ls --help``
return value -- `STDOUT` of the program collected to a String
`%x[]` is the exact same thing

#### Process#spawn
- is a non-blocking call. (`Kernel#system` is a blocking call).

takes many options that allow you to control the behaviour of the child process.

#### IO.popen
Implementing pipes in pure ruby.
It still does `fork` + `exec` underneath.

`IO.popen('ls')` - returns a file descriptor.

It also sets up a pipe to communicate with the spawned process.
```ruby
IO.popen('less', 'w') { |stream|
  # stream is an IO object
  stream.puts "some\ndata"
}
```

#### open3
Open3 allows simultaneous access to the STDIN, STDOUT, and STDERR of a spawned
process.
```ruby
# This is available as part of the standard library.
require 'open3'
Open3.popen3('grep', 'data') { |stdin, stdout, stderr|
  stdin.puts "some\ndata"
  stdin.close
  puts stdout.read
}
```

### In the Real World
One common drawback of all methods: forking, and so using lots of memory.
Another way - `posix-spawn`.

### System Calls
`Kernel#system` - system
`Kernel#exec` - `execve`
`IO.popen` - `popen`
`posix-spawn` - `posix_spawn`

## Chapter 20. Ending
### Abstraction
A kernel has an extremely abstract and simple view of its processes.
All processes look the same to the kernel.
Unix programming is programming language agnostic.

### Communication
The kernel provides very abstract ways of communicating between processes.

## Chapter 21. Appendix: How Resque Manages Processes
## Chapter 22. Unicorn
At a very high level Unicorn is a pre-forking web server.
You tell unicorn how many processes you would like to have.
It initializes network sockets and loads an app.
Then uses fork to create the worker processes.
The master process will keep track of the worker processes, see if they have an ok response time, etc.

Waits for any process, second option makes the call non-blocking.
```ruby
wpid, status = Process.waitpid2(-1, Process::WNOHANG)
```

## Chapter 23 Preforking Servers
### 1. Efficient use of memory
Preforking uses memory more efficiently than does spawning multiple unrelated
processes.

### 2. Efficient Load Balancing
Typically the workflow happens in the same process: A socket is opened, the process waits for connections on that socket. The connection is handled, closed, and the loop starts over again.
Preforking servers: the master process opens a socket before loading the app.
The master process doesn't accept connections.
Forked processes will get a copy of that socket, they accept connections.
The kernel balances the load.

### 3. Efficient Sysandmining
Easier administrating (by a human).

### Basic Example
`Process.waitall` - runs a loop waiting for all child processes to exit and returns array of process statuses.

#### Questions [ru]
- что такое файловые дескрипторы?
- как узнать его в руби?
- как присваивается номер?
- почему номер присваивается не начиная с 0, чем заняты первые номера?
- какие стандартные потоки/ресурсы?
- какая от них польза?
- какие лимиты есть у процессов? можно ли их менять?
- для чего нужны env-переменные
- env - это хэш?
- как парсить аргументы?
- способы завершить процесс
- что будет с дочерними процессами, если родительский процесс завершится?
- что такое CoW, какие плюсы от этого при форке?
- fork в ruby CoW-friendly или нет?
- Зачем нужны `Process.wait` and `waitpid2`?
- что такое зомби-процессы? Как их избежать?
- что такое ipc и какие основные способы?
- чем отличаются pipes от sockets?
- что если нужно ipc между разными машинами?
- что такое session groups и process groups
- что происходит при демонизировании процесса?
- что такое процесс-демон?
- как spawnуть процесс?
- чем отличается system от exec и от ``
- чем отличается `Process.spawn` от них?
- rescue fork
- как unicorn запускает и убивает процессы?
- какие преимущества preforking servers? (по сравнению с non-preforking, e.g. Mongrel)

