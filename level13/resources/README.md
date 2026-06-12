## level 13

For `level 13` there is another binary to break:

```bash
level13@SnowCrash:~$ ls -la
total 20
dr-x------ 1 level13 level13  120 Mar  5  2016 .
d--x--x--x 1 root    users    340 Aug 30  2015 ..
-r-x------ 1 level13 level13  220 Apr  3  2012 .bash_logout
-r-x------ 1 level13 level13 3518 Aug 30  2015 .bashrc
-r-x------ 1 level13 level13  675 Apr  3  2012 .profile
-rwsr-sr-x 1 flag13  level13 7303 Aug 30  2015 level13
```

If we run it we can see the output:

```bash
level13@SnowCrash:~$ ./level13 
UID 2013 started us but we we expect 4242
```

So lets try attaching GDB to it and inspect it a bit more thoroughly.

```
(gdb) disassemble main
Dump of assembler code for function main:
   0x0804858c <+0>:	push   %ebp
   0x0804858d <+1>:	mov    %esp,%ebp
   0x0804858f <+3>:	and    $0xfffffff0,%esp
   0x08048592 <+6>:	sub    $0x10,%esp
   0x08048595 <+9>:	call   0x8048380 <getuid@plt>
   0x0804859a <+14>:	cmp    $0x1092,%eax
   0x0804859f <+19>:	je     0x80485cb <main+63>
   0x080485a1 <+21>:	call   0x8048380 <getuid@plt>
   0x080485a6 <+26>:	mov    $0x80486c8,%edx
   0x080485ab <+31>:	movl   $0x1092,0x8(%esp)
   0x080485b3 <+39>:	mov    %eax,0x4(%esp)
   0x080485b7 <+43>:	mov    %edx,(%esp)
   0x080485ba <+46>:	call   0x8048360 <printf@plt>
   0x080485bf <+51>:	movl   $0x1,(%esp)
   0x080485c6 <+58>:	call   0x80483a0 <exit@plt>
   0x080485cb <+63>:	movl   $0x80486ef,(%esp)
   0x080485d2 <+70>:	call   0x8048474 <ft_des>
   0x080485d7 <+75>:	mov    $0x8048709,%edx
   0x080485dc <+80>:	mov    %eax,0x4(%esp)
   0x080485e0 <+84>:	mov    %edx,(%esp)
   0x080485e3 <+87>:	call   0x8048360 <printf@plt>
   0x080485e8 <+92>:	leave
```

## Solution

If we inspect the following lines:

```
   0x08048595 <+9>:	call   0x8048380 <getuid@plt>
   0x0804859a <+14>:	cmp    $0x1092,%eax
   0x0804859f <+19>:	je     0x80485cb <main+63>
```

We can infer that the the result from `getuid` gets compared to the hex value `0x1092` which is `4242` in decimal.
So if we were to change the return from `getuid` to `4242` by changing the contents of the `eax` register at the right time we should hit the `je` call!

Lets try setting a breakpoint right before the `cmp` call happens:

```
(gdb) b *0x804859a
Breakpoint 1 at 0x804859a
```

And run till we hit it:

```
(gdb) run
Starting program: /home/user/level13/level13 

Breakpoint 1, 0x804859a in main ()
```

We can check the values of our registers using `info registers` or `i r`:

```
(gdb) i r
eax            0x7dd    2013
```
Then change our `eax` register to our needed value of `4242` by using the `set` command:

```
(gdb) set $eax=4242
```

Once again we can confirm the change by running `i r`:

```
(gdb) i r
eax            0x1092   4242
```

And then all there is left to do is `continue` our program:

```
(gdb) c
c
Continuing.
your token is 2A31L79asukciNyi8uppkEuSx
```

And we're presented with our flag: `2A31L79asukciNyi8uppkEuSx`
