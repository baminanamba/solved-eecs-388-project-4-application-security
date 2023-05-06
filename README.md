Download Link: https://assignmentchef.com/product/solved-eecs-388-project-4-application-security
<br>



Intro to Computer Security

<h1>Project 4: Application Security</h1>

work will not be accepted after 19.5 hours past the deadline. If you have a conflict due to travel, interviews, etc., please plan accordingly and turn in your project early.

This is a group project; you will work in teams of two and submit one project per team. Please find a partner as soon as possible. If you have trouble forming a team, post to Piazza’s partner search forum.

The code and other answers your group submits must be entirely your own work, and you are bound by the Honor Code. You may consult with other students about the conceptualization of the project and the meaning of the questions, but you may not look at any part of someone else’s solution or collaborate with anyone outside your group. You may consult published references, provided that you appropriately cite them (e.g., with program comments), as you would in an academic paper.

Solutions must be submitted electronically via Canvas, following the submission checklist below. Please coordinate carefully with your partner to make sure at least one of you submits on time.

<h1>Introduction</h1>

This project will introduce you to control-flow hijacking vulnerabilities in application software, including buffer overflows. We will provide a series of vulnerable programs and a virtual machine environment in which you will develop exploits.

<h2>Objectives</h2>

<ul>

 <li>Be able to identify and avoid buffer overflow vulnerabilities in native code</li>

 <li>Understand the severity of buffer overflows and the necessity of standard defenses</li>

 <li>Gain familiarity with machine architecture and assembly language</li>

</ul>

<h2>Read this First</h2>

This project asks you to develop attacks and test them in a virtual machine you control. Attempting the same kinds of attacks against others’ systems without authorization is prohibited by law and university policies and may result in <em>fines, expulsion, and jail time</em>. You must not attack anyone else’s system without authorization! Per the course ethics policy, you are required to respect the privacy and property rights of others at all times, <em>or else you will fail the course. </em>See the “Ethics, Law, and University Policies” section on the course website.

<h1>Setup</h1>

Buffer-overflow exploitation depends on specific details of the target system, so we are providing an Ubuntu VM in which you should develop and test your attacks. We’ve also slightly tweaked the configuration to disable security features that are commonly used in the wild but would complicate your work. We’ll use this precise configuration to grade your submissions, so do not use your own VM instead.

<ol>

 <li>Download VirtualBox from <a href="https://www.virtualbox.org/">https://www.virtualbox.org/</a> and install it on your computer. VirtualBox runs on Windows, Linux, and Mac OS.</li>

 <li>Get the VM file at <a href="https://eecs388.org/*/appsec-vm.ova">https://eecs388.org/*/appsec-vm.ova.</a> This file is 2 GB, so we recommend downloading it from campus.</li>

 <li>Launch VirtualBox and select File B Import Appliance to add the VM.</li>

 <li>Start the VM. A username and password are not required to login, but if needed they are ubuntu and ubuntu.</li>

 <li>Download <a href="https://eecs388.org/*/targets.tar.gz">https://eecs388.org/*/targets.tar.gz</a> from inside the VM. This file contains the target programs you will exploit.</li>

 <li>tar -xf targets.tar.gz</li>

 <li>cd targets/</li>

 <li>Each group’s targets will be slightly different. Personalize the targets by running:</li>

</ol>

./setcookie <em>uniqname1 uniqname2 </em>Make sure both uniqnames are correct!

<ol start="9">

 <li>sudo make (The password you’re prompted for is ubuntu.)</li>

</ol>

<h1>Resources and Guidelines</h1>

No Attack Tools!         You may not use special-purpose tools meant for testing security or exploiting vulnerabilities. You must complete the project using only general purpose tools, such as gdb.

Control Hijacking Before you begin this project, review the lecture slides from the controlhijacking lecture and attend discussion for additional details. Read “Smashing the Stack for Fun and Profit,” available at <a href="https://eecs388.org/*/stack_smashing.pdf">https://eecs388.org/*/stack_smashing.pdf.</a>

GDB You will make extensive use of the GDB debugger, which you should recall from EECS 280. Useful commands that you may not know are “disassemble”, “info reg”, “x”, and “stepi”. See the GDB help for details, and don’t be afraid to experiment! This quick reference may also be useful: <a href="https://eecs388.org/*/gdb-refcard.pdf">https://eecs388.org/*/gdb-refcard.pdf.</a>

x86 Assembly These are many good references for Intel assembly language, but note that this project targets the 32-bit x86 ISA. The stack is organized differently in x86 and x86_64. If you are reading any online documentation, ensure that it is based on the x86 architecture, not x86_64.

<h1>Targets</h1>

The target programs for this project are simple, short C programs with (mostly) clear security vulnerabilities. We have provided source code and a Makefile that compiles all the targets. Your exploits must work against the targets as compiled and executed within the provided VM.

<h2>target0: Overwriting a variable on the stack                                              (<em>Difficulty: Easy</em>)</h2>

This program takes input from stdin and prints a message. Your job is to provide input that causes the program to output: “Hi <em>uniqname</em>! Your grade is A+.” (You can use either group member’s uniqname.) To accomplish this, your input will need to overwrite another variable stored on the stack.

Here’s one approach you might take:

<ol>

 <li>Examine target0.c. Where is the buffer overflow?</li>

 <li>Start the debugger (gdb target0) and disassemble _main: (gdb) disas _main. Identify the function calls and the arguments passed to them.</li>

 <li>Draw a picture of the stack. How are name[] and grade[] stored relative to each other?</li>

 <li>How could a value read into name[] affect the value contained in grade[]? Test your hypothesis by running ./target0 on the command line with different inputs.</li>

</ol>

What to submit          Create a Python program named sol0.py that prints a line to be passed as input to the target. Test your program with the command line:

python sol0.py | ./target0

Hint: In Python, you can write strings containing non-printable ASCII characters by using the escape sequence “x<em>nn </em>”, where <em>nn </em>is a 2-digit hex value. To cause Python to repeat a character <em>n </em>times, you can do: print “X”*n.

<h2>target1: Overwriting the return address                                                     (<em>Difficulty: Easy</em>)</h2>

This program takes input from stdin and prints a message. Your job is to provide input that makes it output: “Your grade is perfect.” Your input will need to overwrite the return address so that the function vulnerable() transfers control to print_good_grade() when it returns.

<ol>

 <li>Examine target1.c. Where is the buffer overflow?</li>

 <li>Disassemble print_good_grade. What is its starting address?</li>

 <li>Set a breakpoint at the beginning of vulnerable and run the program.</li>

</ol>

(gdb) break vulnerable

(gdb) run

<ol start="4">

 <li>Disassemble vulnerable and draw the stack. Where is input[] stored relative to %ebp? How long would an input have to be to overwrite this value and the return address?</li>

 <li>Examine the %esp and %ebp registers: (gdb) info reg</li>

 <li>What are the current values of the saved frame pointer and return address from the stack frame? You can examine two words of memory at %ebp using: (gdb) x/2wx $ebp</li>

 <li>What should these values be in order to redirect control to the desired function?</li>

</ol>

What to submit          Create a Python program named sol1.py that prints a line to be passed as input to the target. Test your program with the command line:

python sol1.py | ./target1

When debugging your program, it may be helpful to view a hex dump of the output. Try this: python sol1.py | hd

Remember that x86 is little endian. Use Python’s struct module to output little-endian values:

from struct import pack print pack(“&lt;I”, 0xDEADBEEF)

<h2>target2: Redirecting control to shellcode                                                    (<em>Difficulty: Easy</em>)</h2>

The remaining targets are owned by the root user and have the suid bit set. Your goal is to cause them to launch a shell, which will therefore have root privileges. This and targets all take input as command-line arguments rather than from stdin. Unless otherwise noted, you should use the shellcode we have provided in shellcode.py. Successfully placing this shellcode in memory and setting the instruction pointer to the beginning of the shellcode (e.g., by returning or jumping to it)

will open a shell.

<ol>

 <li>Examine target2.c. Where is the buffer overflow?</li>

 <li>Create a Python program named sol2.py that outputs the provided shellcode:</li>

</ol>

from shellcode import shellcode print shellcode

<ol start="3">

 <li>Set up the target in GDB using the output of your program as its argument:</li>

</ol>

gdb –args ./target2 $(python sol2.py)

<ol start="4">

 <li>Set a breakpoint in vulnerable and start the target.</li>

 <li>Disassemble vulnerable. Where does buf begin relative to %ebp? What’s the current value of %ebp? What will be the starting address of the shellcode?</li>

 <li>Identify the address after the call to strcpy and set a breakpoint there:</li>

</ol>

(gdb) break *0x08048efb

Continue the program until it reaches that breakpoint.

(gdb) cont

<ol start="7">

 <li>Examine the bytes of memory where you think the shellcode is to confirm your calculation:</li>

</ol>

(gdb) x/32bx 0x<em>address</em>

<ol start="8">

 <li>Disassemble the shellcode: (gdb) disas/r 0x<em>address</em>,+32 How does it work?</li>

 <li>Modify your solution to overwrite the return address and cause it to jump to the beginning of the shellcode.</li>

</ol>

What to submit Create a Python program named sol2.py that prints a line to be used as the command-line argument to the target. Test your program with the command line: ./target2 $(python sol2.py)

If you are successful, you will see a root shell prompt (#). Running whoami will output “root”.

If your program segfaults, you can examine the state at the time of the crash using GDB with the core dump: gdb ./target2 core. The file core won’t be created if a file with the same name already exists. Also, since the target runs as root, you will need to run it using sudo ./target2 in order for the core dump to be created.

<h2>target3: Overwriting the return address indirectly                            (<em>Difficulty: Medium</em>)</h2>

In this target, the buffer overflow is restricted and cannot directly overwrite the return address. You’ll need to find another way. Your input should cause the provided shellcode to execute and open a root shell.

What to submit Create a Python program named sol3.py that prints a line to be used as the command-line argument to the target. Test your program with the command line:

./target3 $(python sol3.py)

<h2>target4: Beyond strings                                                                               (<em>Difficulty: Medium</em>)</h2>

This target takes as its command-line argument the name of a data file it will read. The file format is a 32-bit count followed by that many 32-bit integers. Create a data file that causes the provided shellcode to execute and opens a root shell.

What to submit Create a Python program named sol4.py that outputs the contents of a data file to be read by the target. Test your program with the command line: python sol4.py &gt; tmp; ./target4 tmp

<h2>target5: Bypassing DEP                                                                                (<em>Difficulty: Medium</em>)</h2>

This program resembles target2, but it has been compiled with data execution prevention (DEP) enabled. DEP means that the processor will refuse to execute instructions stored on the stack. You can overflow the stack and modify values like the return address, but you can’t jump to any shellcode you inject. You need to find another way to run the command /bin/sh and open a root shell.

What to submit Create a Python program named sol5.py that prints a line to be used as the command-line argument to the target. Test your program with the command line:

./target5 $(python sol5.py)

For this target, it’s acceptable if the program segfaults after the root shell is closed.

<h2>target6: Variable stack position                                                                (<em>Difficulty: Medium</em>)</h2>

When we constructed the previous targets, we ensured that the stack would be in the same position every time the vulnerable function was called, but this is often not the case in real targets. In fact, a defense called ASLR (address-space layout randomization) makes buffer overflows harder to exploit by changing the starting location of the stack and other memory areas on each execution. This target resembles target2, but the stack position is randomly offset by 0–255 bytes each time it runs. You need to construct an input that always opens a root shell despite this randomization.

What to submit Create a Python program named sol6.py that prints a line to be used as the command-line argument to the target. Test your program with the command line:

./target6 $(python sol6.py)

A word of caution       If you see any output from the program before a root shell is opened, you have not done target6 of the project correctly and your solution will not be accepted by the autograder.

<h2>target7: Heap-based exploitation [Extra credit]                                       (<em>Difficulty: Hard</em>)</h2>

This program implements a doubly linked list on the heap. It takes three command-line arguments. Figure out a way to exploit it to open a root shell. You may need to modify the provided shellcode slightly.

What to submit          Create a Python program named sol7.py that prints lines to be used for each of the command-line arguments to the target. Your program should take a single numeric argument that determines which of the three arguments it outputs. Test your program with the command line:

./target7 $(python sol7.py 1) $(python sol7.py 2) $(python sol7.py 3) target8: Return-oriented programming [Extra credit] (<em>Difficulty: Hard</em>)

This target is identical to target2, but it is compiled with DEP enabled. Implement a ROP-based attack to bypass DEP and open a root shell.

What to submit Create a Python program named sol8.py that prints a line to be used as the command-line argument to the target. Test your program with the command line:

./target8 $(python sol8.py)

You may find the objdump utility helpful.

For this target, it’s acceptable if the program segfaults after the root shell is closed.

<h2>target9: Callback shell [Extra credit]                                                             (<em>Difficulty: Hard</em>)</h2>

This target uses the same code as target3, but you have a different objective. Instead of opening a root shell, implement your own shellcode to implement a <em>callback shell</em>. Your shellcode should open a TCP connection to 127.0.0.1 on port 31337. Commands received over this connection should be executed in a shell, and the output should be sent back to the remote machine.

What to submit Create a Python program named sol9.py that prints a line to be used as the command-line argument to the target. Test your program with the command line:

./target9 $(python sol9.py)

For the remote end of the connection, use netcat: nc -l 31337

To receive credit, you must include (as an extended comment in your Python file) a fully annotated disassembly on your shellcode that explains in detail how it works.

<h1>Fuzz Testing</h1>

Manually reviewing source code for vulnerabilities can be laborious and time consuming, and outsiders typically cannot do it at all for closed-source software. For these reasons, both attackers and defenders often use an automated form of vulnerability discovery called “fuzz testing” or

“fuzzing” that attempts to find edge-cases that the application developers failed to account for. Unlike analysis that makes use of source code (“white-box testing”), fuzzing assumes only the ability to execute the software with chosen inputs (“black-box testing”).

In fuzzing, the analyst creates a program (a “fuzzer”) that emulates a user and rapidly provides many different automatically generated inputs to the target application while monitoring for anomalous behavior (e.g., crashes or corrupted return data). When an input consistently causes anomalous behavior, the fuzzer stores it so that the analyst can investigate the problem. The anomalous behavior may be a sign that there is an exploitable vulnerability in the code path that the input exercises.

It’s not usually feasible to test with every possible input, but a clever input generation algorithm can increase the odds that the fuzzer will trigger a bug. For instance, many fuzzers start with a set of valid inputs and then corrupt them by making randomized changes, additions, or deletions.

Setup  A recently founded start-up named ZCorp has hit it big by providing data analysis expertise to customers. Customers provide data to analyze in JSON format, and ZCorp charges them based on the number of JSON string values in each customer’s data. In order to generate bills, they run all customer input through a JSON parser that extracts each JSON string value, along with its index.

ZCorp outsourced development of this JSON parser to an acquaintance of one of the founders. Unfortunately for ZCorp, this developer never took EECS 388, so the parser is probably highly vulnerable to exploitation. If you can find an input that causes a SEGFAULT in the parser, ZCorp can refuse to pay the inept developer until the problem has been fixed.

Goal Your goal for this portion of the project is to create a fuzzer that is capable of automatically finding an input that causes a SEGFAULT in the provided JSON parser. You can download the jsonParser binary from <a href="https://eecs388.org/*/parser.tar.gz">https://eecs388.org/*/parser.tar.gz.</a> It reads JSON data from stdin and writes to stdout and stderr. While the developer did not provide ZCorp with the source code for the parser, they did provide a script, jsonParserTests.py, that checks a set of test cases.

You are <u>not </u>required to create an exploit or understand the cause of the SEGFAULT.

You should <u>not </u>attempt to reverse-engineer the target, as this will likely be a waste of time.

What to submit Create a Python program named fuzzer.py that generates inputs, invokes jsonParser on them, and then determines whether there was a SEGFAULT. Your program should do this repeatedly for different inputs until a SEGFAULT occurs. When this happens, it should print the input data that triggered the fault to stdout as a base64-encoded string and exit. You can confirm that the input causes a SEGFAULT via the command:

echo “<em>base64 encoded data</em>” | base64 -d | ./jsonParser

When you find an input that consistently causes a SEGFAULT, place the base64-encoded string in a text file named fuzzInput.txt and submit it along with your program.

<h1>Submission Checklist</h1>

Upload to Canvas a gzipped tar file named project4.<em>uniqname1</em>.<em>uniqname2</em>.tar.gz that contains only the files listed below. These will be autograded, so make sure you have the proper filenames, formats, and behaviors. Failure to work with the autograder—for any reason—will result in a 5% deduction from the maximum possible points. You can generate the tarball at the shell using this command:

tar -zcf project4.<em>uniqname1</em>.<em>uniqname2</em>.tar.gz cookie sol[0123456789].py  fuzzer.py fuzzInput.txt The tarball should contain only the files below:

cookie [Generated by setcookie based on your uniqnames.] sol0.py sol1.py sol2.py sol3.py sol4.py sol5.py sol6.py

sol7.py [Optional extra credit.] sol8.py [Optional extra credit.] sol9.py [Optional extra credit.] fuzzer.py fuzzInput.txt

Your files can make use of standard Python libraries and the provided shellcode.py, but they must be otherwise self-contained. Do not include shellcode.py with your submission. Be sure to test that your solutions work correctly in the provided VM without installing any additional packages.