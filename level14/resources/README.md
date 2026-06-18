## level 14

Starting off `level 14` we aren't able to see anything out of the ordinary:

```bash
level14@SnowCrash:~$ ls -la
total 12
dr-x------ 1 level14 level14  100 Mar  5  2016 .
d--x--x--x 1 root    users    340 Aug 30  2015 ..
-r-x------ 1 level14 level14  220 Apr  3  2012 .bash_logout
-r-x------ 1 level14 level14 3518 Aug 30  2015 .bashrc
-r-x------ 1 level14 level14  675 Apr  3  2012 .profile
```

And after a couple of hours of searching we couldn't find anything that could result in a privilege escalation, so all we're left with is the command we have been using since the very start: `getflag`.


## Solution

After having learned a bit of GDB in the previous exercise we decided to try and make use of it again, despite the program being 'defended' against it using a `ptrace` call:

```
(gdb) run
Starting program: /bin/getflag 
You should not reverse this
[Inferior 1 (process 2967) exited with code 01]
```

The challenge for this one will be breaking through this protection!
So lets start off with reading the disassembly of the main:

```asm
(gdb) disas main
Dump of assembler code for function main:
   0x08048946 <+0>:	push   %ebp
   0x08048947 <+1>:	mov    %esp,%ebp
   0x08048949 <+3>:	push   %ebx
   0x0804894a <+4>:	and    $0xfffffff0,%esp
   0x0804894d <+7>:	sub    $0x120,%esp
   0x08048953 <+13>:	mov    %gs:0x14,%eax
   0x08048959 <+19>:	mov    %eax,0x11c(%esp)
   0x08048960 <+26>:	xor    %eax,%eax
   0x08048962 <+28>:	movl   $0x0,0x10(%esp)
   0x0804896a <+36>:	movl   $0x0,0xc(%esp)
   0x08048972 <+44>:	movl   $0x1,0x8(%esp)
   0x0804897a <+52>:	movl   $0x0,0x4(%esp)
   0x08048982 <+60>:	movl   $0x0,(%esp)
   0x08048989 <+67>:	call   0x8048540 <ptrace@plt> <----- evil ptrace call!
   0x0804898e <+72>:	test   %eax,%eax
   0x08048990 <+74>:	jns    0x80489a8 <main+98>
   0x08048992 <+76>:	movl   $0x8048fa8,(%esp)
   0x08048999 <+83>:	call   0x80484e0 <puts@plt>
   0x0804899e <+88>:	mov    $0x1,%eax
   0x080489a3 <+93>:	jmp    0x8048eb2 <main+1388>
   { ... }
```

Here we can see that `ptrace` is being called, and then it's return is checked to see if any programs have already attached themselves to this binary. Only one program can attach itself at a time, since `gdb` has already done so the call to `ptrace` will fail. We can simply pretend that it went through by changing the return value once again inside of the `$eax` register!

```
(gdb) break *main+72
Breakpoint 1 at 0x804898e
```

Now we can run our program and change the value of `$eax` to 0, to pretend that the `ptrace` call had no issues!

```
(gdb) r
Starting program: /bin/getflag

Breakpoint 1, 0x0804898e in main ()
(gdb) i r
eax            0xffffffff	-1
ecx            0xb7e2b900	-1209878272
edx            0xffffffc8	-56
ebx            0xb7fd0ff4	-1208152076
esp            0xbffff4e0	0xbffff4e0
ebp            0xbffff608	0xbffff608
esi            0x0	0
edi            0x0	0
eip            0x804898e	0x804898e <main+72>
eflags         0x200282	[ SF IF ID ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x33	51
(gdb) set %eax=0
A syntax error in expression, near `%eax=0'.
(gdb) set $eax=0
(gdb) i r
eax            0x0	0
ecx            0xb7e2b900	-1209878272
edx            0xffffffc8	-56
ebx            0xb7fd0ff4	-1208152076
esp            0xbffff4e0	0xbffff4e0
ebp            0xbffff608	0xbffff608
esi            0x0	0
edi            0x0	0
eip            0x804898e	0x804898e <main+72>
eflags         0x200282	[ SF IF ID ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x33	51
```

Next up we're going to have to pretend to be the `flag14` user. If we go back to the disassembly of main we can see the following:

```
{ ... }
   0x08048af8 <+434>:	call   0x80484c0 <fwrite@plt>
   0x08048afd <+439>:	call   0x80484b0 <getuid@plt> <----------- getuid call
   0x08048b02 <+444>:	mov    %eax,0x18(%esp)
   0x08048b06 <+448>:	mov    0x18(%esp),%eax
   0x08048b0a <+452>:	cmp    $0xbbe,%eax 
   0x08048b0f <+457>:	je     0x8048ccb <main+901>
   0x08048b15 <+463>:	cmp    $0xbbe,%eax
   0x08048b1a <+468>:	ja     0x8048b68 <main+546>
   0x08048b1c <+470>:	cmp    $0xbba,%eax
   0x08048b21 <+475>:	je     0x8048c3b <main+757>
   0x08048b27 <+481>:	cmp    $0xbba,%eax
   0x08048b2c <+486>:	ja     0x8048b4d <main+519>
   0x08048b2e <+488>:	cmp    $0xbb8,%eax
   0x08048b33 <+493>:	je     0x8048bf3 <main+685>
   0x08048b39 <+499>:	cmp    $0xbb8,%eax
   0x08048b3e <+504>:	ja     0x8048c17 <main+721>
   0x08048b44 <+510>:	test   %eax,%eax
   0x08048b46 <+512>:	je     0x8048bc6 <main+640>
   0x08048b48 <+514>:	jmp    0x8048e06 <main+1216>
   0x08048b4d <+519>:	cmp    $0xbbc,%eax
   0x08048b52 <+524>:	je     0x8048c83 <main+829>
   0x08048b58 <+530>:	cmp    $0xbbc,%eax
   0x08048b5d <+535>:	ja     0x8048ca7 <main+865>
   0x08048b63 <+541>:	jmp    0x8048c5f <main+793>
   0x08048b68 <+546>:	cmp    $0xbc2,%eax
   0x08048b6d <+551>:	je     0x8048d5b <main+1045>
   0x08048b73 <+557>:	cmp    $0xbc2,%eax
   0x08048b78 <+562>:	ja     0x8048b95 <main+591>
   0x08048b7a <+564>:	cmp    $0xbc0,%eax
   0x08048b7f <+569>:	je     0x8048d13 <main+973>
   0x08048b85 <+575>:	cmp    $0xbc0,%eax
   0x08048b8a <+580>:	ja     0x8048d37 <main+1009>
   0x08048b90 <+586>:	jmp    0x8048cef <main+937>
   0x08048b95 <+591>:	cmp    $0xbc4,%eax
   0x08048b9a <+596>:	je     0x8048da3 <main+1117>
   0x08048ba0 <+602>:	cmp    $0xbc4,%eax
   0x08048ba5 <+607>:	jb     0x8048d7f <main+1081>
   0x08048bab <+613>:	cmp    $0xbc5,%eax
   0x08048bb0 <+618>:	je     0x8048dc4 <main+1150>
   0x08048bb6 <+624>:	cmp    $0xbc6,%eax <----------- Compare we want to hit
   0x08048bbb <+629>:	je     0x8048de5 <main+1183>
   0x08048bc1 <+635>:	jmp    0x8048e06 <main+1216>
   0x08048bc6 <+640>:	mov    0x804b060,%eax
   0x08048bcb <+645>:	mov    %eax,%edx
   0x08048bcd <+647>:	mov    $0x8049090,%eax
   0x08048bd2 <+652>:	mov    %edx,0xc(%esp)
   0x08048bd6 <+656>:	movl   $0x21,0x8(%esp)
   0x08048bde <+664>:	movl   $0x1,0x4(%esp)
   0x08048be6 <+672>:	mov    %eax,(%esp)
   0x08048be9 <+675>:	call   0x80484c0 <fwrite@plt>
   0x08048bee <+680>:	jmp    0x8048e2f <main+1257>
   0x08048bf3 <+685>:	mov    0x804b060,%eax
   0x08048bf8 <+690>:	mov    %eax,%ebx
   { ... }
```

We can see a call to `getuid` followed by a bunch of `cmp` and `je` calls.
Lets first figure out which user ID we need:

```bash
level14@SnowCrash:~$ id flag14
uid=3014(flag14) gid=3014(flag14) groups=3014(flag14),1001(flag)
```

The `flag14` user has the uid of `3014`, converting that to hex we get `0xbc6`.
We now have all the tools we need! Lets change the return of `getuid`:

```
(gdb) break *main+452
Breakpoint 2 at 0x8048b0a
(gdb) cont
Continuing.

Breakpoint 2, 0x08048b0a in main ()
(gdb) info r
eax            0x7de	2014
ecx            0xb7fda000	-1208115200
edx            0x20	32
ebx            0xb7fd0ff4	-1208152076
esp            0xbffff4e0	0xbffff4e0
ebp            0xbffff608	0xbffff608
esi            0x0	0
edi            0x0	0
eip            0x8048b0a	0x8048b0a <main+452>
eflags         0x200246	[ PF ZF IF ID ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x33	51
(gdb) set $eax=3014
(gdb) info r
eax            0xbc6	3014
ecx            0xb7fda000	-1208115200
edx            0x20	32
ebx            0xb7fd0ff4	-1208152076
esp            0xbffff4e0	0xbffff4e0
ebp            0xbffff608	0xbffff608
esi            0x0	0
edi            0x0	0
eip            0x8048b0a	0x8048b0a <main+452>
eflags         0x200246	[ PF ZF IF ID ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x33	51
```

So after having gone through all these steps we can continue running gdb:

```
(gdb) continue
Continuing.
Check flag.Here is your token : 7QiHafiNa3HVozsaXkawuYrTstxbpABHD8CPnHJ
[Inferior 1 (process 2527) exited normally]
(gdb) YAY
```

And we're presented with the final flag: `7QiHafiNa3HVozsaXkawuYrTstxbpABHD8CPnHJ`
