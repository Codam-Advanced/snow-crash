## level 09

Once again, new level new `ls`!

```bash
level09@SnowCrash:~$ ll
total 24
dr-x------ 1 level09 level09  140 Mar  5  2016 ./
d--x--x--x 1 root    users    340 Aug 30  2015 ../
-r-x------ 1 level09 level09  220 Apr  3  2012 .bash_logout*
-r-x------ 1 level09 level09 3518 Aug 30  2015 .bashrc*
-r-x------ 1 level09 level09  675 Apr  3  2012 .profile*
-rwsr-sr-x 1 flag09  level09 7640 Mar  5  2016 level09*
----r--r-- 1 flag09  level09   26 Mar  5  2016 token
```
We are able to see a `token` file and a `level09` executable.
Trying to read the `token` file results in a bunch of garbage:

```bash
level09@SnowCrash:~$ cat token 
f4kmm6p|=�p�n��DB�Du{��
```

And running the executable gives us the following:

```bash
level09@SnowCrash:~$ ./level09 
You need to provied only one arg.
level09@SnowCrash:~$ ./level09 token
tpmhr
```

Since we still wanted to stay away from ghidra and we already tried the strings command before we wanted to attempt a different approach, a debugger.
But after attempting to attach GDB to the executable we were greeted with a nice warning:

```bash
(gdb) run
Starting program: /home/user/level09/level09 
You should not reverse this
```

## Solution

After a couple of attempts we noticed that the output changed depending on the argument given and that it does not read a file.
And after handing it a bunch of 0's we noticed a pattern:

```bash
level09@SnowCrash:~$ ./level09 00
01
level09@SnowCrash:~$ ./level09 000000000000000
0123456789:;<=>
```

The given input gets changed in a specific way, the index of the character is added to the ascii value.
So to reverse this all we would have to do is subtract the index of the character from its value!

Let's make a small C file that does just that:

```bash
level09@SnowCrash:~$ vim reverse.c
```

```c
#include <unistd.h>

int main(int argc, char **argv)
{
	if (argc != 2)
		return 1;

	char *input = argv[1];

	for (int i = 0; input[i]; i++)
	{
		input[i] -= i;
		write(1, &input[i], 1);
	}

	write(1, "\n", 1);
	return 0;
}
```

After saving, compiling and running it with the contents of the `token` file we see the following:


```bash
level09@SnowCrash:~$ gcc reverse.c -std=c99 -o reverse
level09@SnowCrash:~$ ./reverse $(cat token)
f3iji1ju5yuevaus41q1afiuq
```

We seem to have successfully reversed the encryption of the `token`, resulting in `f3iji1ju5yuevaus41q1afiuq`!

Now all that is left to do is retrieve our flag:

```bash
level09@SnowCrash:~$ su flag09
Password: f3iji1ju5yuevaus41q1afiuq
Don't forget to launch getflag !
flag09@SnowCrash:~$ getflag
Check flag.Here is your token : s5cAJpM8ev6XHw998pRWG728z
```

And with the flag `s5cAJpM8ev6XHw998pRWG728z` we have completed level 09!
