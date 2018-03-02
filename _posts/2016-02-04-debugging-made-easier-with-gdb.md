---
id: 525
title: Debugging made easier with GDB
date: 2016-02-04T10:50:43+00:00
author: Robert
layout: post
guid: http://wiredtron.com/?p=525
permalink: /2016/02/04/debugging-made-easier-with-gdb/
categories:
  - Uncategorized
---
*(Authors Note: This was a paper I initially wrote as part of a project for class. I feel it came out well enough that it may also be useful to others.)*

Whenever a computer program is written, something has to be done to the written code in order to convert it into a set of instructions which a computer can run. In order to allow code to be readable by a computer, languages such as C and C++ compile the code using a compiler ("Definition of Compiler"). Code from the aforementioned languages is then compiled into **machine code** (see glossary) (Albert, 2012). Machine code is good for the computer because it is able to then easily understand the set of instructions which the code provides, but those instructions are much harder for a human to interpret. Since coding is such a tedious task, the chances of a program working exactly as intended without any problems the first time it runs is very low. Knowing what is going on behind the scenes and how to attack problems such as **segmentation faults**, **buffer overflows**, and any other issues you may encounter in your program by looking at the low level operations being performed, reading the **assembly code** for your program, and using a debugger such as the **GNU** debugger (GDB) are often the most important parts of creating working software.

**What is GDB?**

So what is GDB, and why is it so useful? Well GDB is a debugger which handles a plethora of different languages, and is very widely used by programmers. In fact, according to Matloff and Salzman (2008) it is the most commonly used debugging tool among Unix developers (p. 2). GDB provides you with the ability to set **watchpoints** (p. 3) and **breakpoints** (p. 6) in your code and even inspect variables in real-time while your code is running on the computer (p. 9). This provides the programmer with an easy way to see what exactly has gone wrong in their program. Before jumping right into debugging with GDB a few prerequisites need to be covered.

**Converting to and from Hexadecimal**

Before we can dive into dealing with GDB, there are a few concepts we have to deal with first. The first of these is that everything at the base level in computers is represented using binary which is base 2. In order to make it easier to read addresses in the computer and such, programmers use base 16 which can easily be converted into from base 2 ("Hexadecimal Computer Code").  This usage of hexadecimal will become evident once you dig into GDB. Since hex is base 16, 6 more single-digit numbers had to be added. Therefore, 10-15 are represented by A-F respectively ("Hexadecimal Computer Code"). Since hex is base 16, this means the only numbers that have the same meaning in hex and decimal are the numbers 1-9. The number 10 in hex is equal to 16 in decimal. In order to convert from hex to decimal you simply raise 16 to the location of the place you are looking at, and then multiply that place by the result ("Hexadecimal Computer Code"). Note that you would raise to the power of 0 for the 1's place, the power of 1 for the 10's place, the power of 2 for the 100's place, and so on ("Hexadecimal Computer Code"). For example 123 converted from hex to decimal would equal 1x16^2^+2x16~­~^1^+3x16^0^=256+32+3=291. In order to reverse the process, you raise 16 to the highest power you can without causing your result to be bigger than your number ("Hexadecimal Computer Code"). Since our number in decimal is 291, this means we can divide it by 16^2^ which is equal to 256. 256 goes into 291 once with a remainder of 35. The highest power of 16 we can fit into 35 is 16^1^ or 16. 16 can go into 35 two times with a remainder of 3. This 3 is our 1's place. Since the number of times each power of 16 could be divided into the original number is the value for that hexadecimal digit, we are left with 1 for the 100's place, 2 for the 10's place, and 3 for the 1's place. If this does not make sense at first, it may be good to keep a hex to decimal calculator handy when diving into GDB.

**The Stack and the Heap**

The stack and the heap are two concepts that apply when programming in C, and because of this it is important to have an understanding of them before attempting to debug a C program. Both the stack and heap are stored in random access memory (RAM) on the computer, however the heap is used to allocate memory on demand whereas the stack has a specific data structure (Bondy, 2008). Due to the specific structure of the stack you are only able to add or remove values on the top of the stack (Bendersky, 2011). It is important to understand that even though it is called the top of the stack it is really the lowest available address in the stack which is allocated to the program (2011). This means that if I add data to the stack then the pointer which points to the top of the stack actually moves downward through memory addresses (2011). It is also important to know that when working with the stack there is a register called the stack pointer that always points to the top of the stack (Albert, 2012).

**A crash course in C**

Before debugging programs using GDB, there is a need for code to debug. In order to make sure everyone is on the same page while reading through later, it seems best to cover the basic elements of a C program. The following table should provide a simple reference guide for later on.

<table width="624">
<tbody>
<tr>
<td width="130"><strong>Name</strong></td>
<td width="279"><strong>Description</strong></td>
<td width="215"><strong>Example</strong></td>
</tr>
<tr>
<td width="130">Comment</td>
<td width="279">A block of code which is not compiled and is only visible in the source for other programmers to read.</td>
<td width="215">/* This is a comment */<p></p>
<p>// This is a one line comment</p></td>
</tr>
<tr>
<td width="130">Variables</td>
<td width="279">Something which holds an assigned value. Note that if these are declared before <strong>compile time</strong> then these variables are all declared in the stack.</td>
<td width="215">int x = 3;<p></p>
<p>char firstLetter = ‘J’;</p></td>
</tr>
<tr>
<td width="130">Commonly Imported functions</td>
<td width="279">Most C programs include the C Standard Input and Output Library stdio.h (“C library to perform”). This provides us with a few functions we can call in order to write data to the screen and do other seemingly basic manipulations.</td>
<td width="215">printf(“hello world!”);</td>
</tr>
<tr>
<td width="130">Main function</td>
<td width="279">This is the function which is called when the program is executed</td>
<td width="215">int main(int argc, char *argv[])<p></p>
<p>{// Program code goes here}</p>
<p>&nbsp;</p></td>
</tr>
</tbody>
</table>

*Data compiled from Shaw (2012).*

These are not the only things which C has to offer, however for the purpose of giving a basic understanding of the topics to come, knowledge of these above terms and examples should be sufficient to carry you through the explanations to come. Please note that any code in this guide is compiled using GCC 4.8.3 which can be downloaded from http://gcc.gnu.org/install/download.html.

**Reading Assembly Code**

Since GDB can be used with and without the source code handy, it is important to cover the topic of just what assembly code is and why it is useful. Before that can happen, a clarification needs to be made. Two of the first things mentioned above were assembly and machine language, however no useful distinction between the two was made. Machine language is the actual set of instructions which the computer can execute on the CPU and can be represented using hexadecimal, while assembly language is a more human-readable version of machine code (Coehoorn, 2012). The translation between machine code and assembly can pretty much be looked at as the machine simply matching the instructions in machine language with their assembly equivalent (2012). There are two different very popular assembly syntaxes, Intel and AT&T ("Using Assembly Language"). Since mixing the two different syntaxes together can cause confusion, in this guide only AT&T syntax will be used.

AT&T syntax is written with the instruction followed by the source and destination separated by a comma. The instruction text also often contains an extra letter designating the type of data it is. A few examples of this are l for long, b for byte, and w for word ("Using Assembly Language"). This is important to keep in mind later on when you start seeing instructions such as addl, which simply means "add long" ("Using Assembly Language"). It is also important to note that **registers** in AT&T syntax are prefixed by a % and **immediate values** are prefixed by a $ (Albert, 2012). There are a large number of instructions available however for the purpose of keeping this introduction gentle the following ones will be used.

| **Instruction** | **Description** |
| add | Adds together two parameters provided and stores the result in the location of the first parameter. This instruction always must have two parameters. |
| push | Push the value of the parameter onto the top of the stack. This instruction only takes one parameter. |
| pop | Take the top parameter from the stack and put it into the specified parameter. This instruction only takes one parameter. |
| mov | Copy the value of the second parameter into the first. This instruction takes two parameters. |
| ret | Marks the end of execution for the current function and returns the thread back to the function which called it. |

*Data compiled from Ferrari, Batson, Lack, Jones, & Evans (2012).*

**Basic GDB Commands**

GDB is contains a huge number of commands which it can run, however for the purpose of this tutorial the list of commands mentioned will be kept to the most popular ones.

| **Command** | **Description** |
| break | Set a breakpoint at a specified function. (In our programs we will mostly break at main since it is our only function.) |
| run | Run the program currently open in GDB. |
| next | Run the next line of the program. This works when you have paused execution with a break or in some other manner. It also takes a parameter which allows you to step more than 1 line at a time. |
| print | Prints out the value of a variable passed in as a parameter. |
| disassemble | Displays the specified function as assembly. |
| x | Examine the specified memory. |
| watch | Watches a variable and only pauses execution of the program is the variable changes. |

*Data compiled from the GDB Manual.*

Please note that in this guide the GDB version being used is 7.6.1, however any version close to this one should work. GDB can be downloaded from http://www.gnu.org/software/gdb/download/.

**Examining a C program in GDB**

Now that the basics have been covered, it is time to create and compile a C program and examine it with GDB. In order to do this you will need to open a new terminal window and create a file. In our case the file can both be created and edited in the same command.

> nano example1.c

This will open the nano text editor and give you a place to type in your code ("GNU nano", 2009). The following code is what will be used for the first debugging example.

> int main(int argc, char *argv[])
>
> {
>
> int number = 16; // Set a variable equal to 16
>
> return 0; // Exit the program
>
> }

In order to exit nano and save hit CTRL+x on the keyboard and then hit y and the enter key ("GNU nano", 2009). In order to compile this program run the command:

> gcc example1.c -g -O0  -o example1

Once you have compiled the program, you can run the program by typing the following command in your current directory.

> ./example1

The program will print out Hello World and then exit. Next it's time to dig in with GDB. In order to do so you will have to run the command:

> gdb example1

This will open an instance of GDB and load our example1 binary. Once GDB has loaded we can take a look at just what our small program has generated in assembly by typing the command:

> disassemble main

This disassembles the main function in our program and prints the assembly code to the screen in AT&T syntax. The result is something like this:

> Dump of assembler code for function main:
>
> 0x00000000004004ed <+0>:          push   %rbp
>
> 0x00000000004004ee <+1>:           mov    %rsp,%rbp
>
> 0x00000000004004f1 <+4>:           mov    %edi,-0x14(%rbp)
>
> 0x00000000004004f4 <+7>:           mov    %rsi,-0x20(%rbp)
>
> 0x00000000004004f8 <+11>:         movl   $0x10,-0x4(%rbp)
>
> 0x00000000004004ff <+18>:         mov    $0x0,%eax
>
> 0x0000000000400504 <+23>:        pop    %rbp
>
> 0x0000000000400505 <+24>:        retq
>
> End of assembler dump.

Based on the earlier discussion of AT&T syntax, some of this should look familiar. Before digging into the assembly code too much, it is a good idea to tread lightly on messing with assembly in GDB until some comfort has been found in using GDB itself. The next step will be to run the following commands in order in GDB.

> break main
>
> run
>
> next

At this point GDB will have stepped us through to the very last instruction of the program. Because of our location at the last command in the program, the print command will work to print out the value of any variable we have declared, because at this point they are all loaded into memory.

> print number

Will print out the value of the number variable declared in our source code, which is 16. Note that the number 16 is pushed into the memory location of the base pointer minus 4 above, however the fact that the value being pushed is 16 is hiding in plain sight. Recall that in the computer, binary is how information is stored, however in order to make it more human readable GDB converts a lot of the information to hexadecimal. So the line

> 0x00000000004004f8 <+11>:           movl   $0x10,-0x4(%rbp)

is read by the computer as "move the hexadecimal value 10 into the the base pointer of the stack minus 4". It is important to mention that 10 is a hexadecimal value, and when converted to decimal it is equal to 16. We can verify that we are correct using the examine command on the memory address which the data was copied into.

> x $rbp-4

Which supplies us with 0x00000010.

**Debugging a program crash**

So how can GDB help to track down the problem if a program keeps crashing? The following example program can be used to find out.

> int main(int argc, char *argv[])
>
> {
>
> int ia[10];
>
> ia[99999999] = 4;
>
> return 0; // Exit the program
>
> }

Save this program as example2.c using the same editor instructions as before. The problem in this program is rather obvious, we are attempting to set a value well outside the range of the array we have declared (Maruseac, 2011), however let us see what GDB can do with this situation. First the program needs to be compiled.

> gcc example2.c -g -O0  -o example2

Next attempt to run the program.

> ./example2

This gives the result

> Segmentation fault (core dumped)

Something has gone wrong, this is a good time to open GDB.

> gdb example2

With GDB running, it is apparent that the instruction telling the computer to move 4 into the base pointer minus our specified offset is:

> 0x00000000004004f8 <+11>:         movl   $0x4,0x17d783cc(%rbp)

So for a start, attempting to look at the memory which we are trying to put data into might give us some clues.

> break main
>
> run
>
> x $rbp-0x17d783cc

When we attempt to access the specified memory address we receive an error.

> 0x7fffe8285b14:         Cannot access memory at address 0x7fffe8285b14

This is the error that is the cause of our segmentation fault. Since we do not have access to this memory, the program is being closed and throwing a segmentation fault to tell us that the program tried to access memory which it does not have permission to access (Maruseac, 2011).

In conclusion, GDB is a very powerful tool which can do many more things than the simple things listed above, however it comes with a bit of a learning curve. The amount of time saved by a programmer by being able to inspect variables on the run and set breakpoints for a function that they are testing, or watchpoints for variables as they change adds up very quickly. Also as the complexity of the program increases, so does the difficulty to debug. All the features of GDB become more useful to the programmer when more difficult concepts are introduced into the program. All of these reasons put together are why GDB is considered the most commonly used debugger by Unix developers.

**Glossary**

-   **Assembly code:** Instructions which are matched in a mostly 1:1 manner with machine code in order to make machine code more readable to a human.
-   **Breakpoint:** A point which the person debugging specifies in the program at which point the debugger will pause execution and allow you to play with the program in the paused state.
-   **Buffer Overflow:** When a program writes outside of its bounds and overwrites other data on the heap or stack.
-   **Compile Time:** The time at which the code for the program is being converted from source code which the user writes into machine code which the computer can read.
-   **GNU:** GNU stands for GNU's not **Unix**. It is a recursive acronym which some programmers find humor in due to the usage of recursion sometimes found in computer software.
-   **Immediate Values:** A value directly supplied to the function.
-   **Machine Code:** Code which is fully compiled and is ready to be executed on the CPU (central processing unit) of the computer. This code is written in binary so it would be very difficult for a human to read.
-   **Registers:** Small amounts of storage avaliable on the CPU for operations.
-   **Segmentation Fault:** When a program requests access to memory that it does not have access to.
-   **Unix:** Unix is officially trademarked as UNIX and is a multi-user environment which is implemented on a variety of different platforms.
-   **Watchpoint:** A watchpoint monitors a variable and only pauses the program if that variable changes.

**References**

Albert, D. (2012, September 12). Understanding C by learning assembly. Hacker School. Retrieved November 11, 2013, from https://www.hackerschool.com/blog/7-understanding-c-by-learning-assembly

Bendersky, E. (2011, February 4). Where the top of the stack is on x86. Eli Benderskys website. Retrieved November 13, 2013, from http://eli.thegreenplace.net/2011/02/04/where-the-top-of-the-stack-is-on-x86/

Bondy, B. (2008, September 17). What and where are the stack and heap. Stack Overflow. Retrieved November 13, 2013, from http://stackoverflow.com/questions/79923/what-and-where-are-the-stack-and-heap

C library to perform Input/Output operations. (n.d.). C++ Reference. Retrieved November 12, 2013, from http://www.cplusplus.com/reference/cstdio/

Coehoorn, J. (2012, August 27). Assembly code vs Machine code vs Object code?. Stack Overflow. Retrieved November 12, 2013, from http://stackoverflow.com/questions/466790/assembly-code-vs-machine-code-vs-object-code

Definition of Compiler. (n.d.). PC Magazine Encyclopedia. Retrieved November 11, 2013, from http://www.pcmag.com/encyclopedia/term/40105/compiler

Ferrari, A., Batson, A., Lack, M., Jones, A., & Evans, D. (2012, September 11). x86 Assembly Guide. Guide to x86 Assembly. Retrieved November 12, 2013, from http://www.cs.virginia.edu/~evans/cs216/guides/x86.html

GDB Manual. (n.d.). *Linux Documentation*. Retrieved September 4, 2013, from http://linux.die.net/man/1/gdb

GNU nano. (2009, November 30). GNU nano. Retrieved November 13, 2013, from http://www.nano-editor.org/

Hexadecimal Computer Code. (n.d.). The Problem Site. Retrieved November 12, 2013, from http://www.theproblemsite.com/codes/hex.asp

Maruseac, M. (2011, August 3). What can cause segmentation faults in C++. What can cause segmentation faults in C++. Retrieved November 13, 2013, from http://stackoverflow.com/questions/6923574/what-can-cause-segmentation-faults-in-c

Matloff, N. S., & Salzman, P. J. (2008). The Art of Debugging with GDB, DDD, and Eclipse. San Francisco: No Starch Press.

Shaw, Z. (2012, January 24). Learn C The Hard Way. Learn C The Hard Way. Retrieved November 12, 2013, from http://c.learncodethehardway.org/book/

Using Assembly Language in Linux. (n.d.). *Linux Assembly*. Retrieved September 4, 2013, from http://asm.sourceforge.net/articles/linasm.html
