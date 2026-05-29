## Solution
Logging in as the flag02 user you are greeted with the current files:
```bash
ll
total 24
dr-x------ 1 level02 level02  120 Mar  5  2016 ./
d--x--x--x 1 root    users    340 Aug 30  2015 ../
-r-x------ 1 level02 level02  220 Apr  3  2012 .bash_logout*
-r-x------ 1 level02 level02 3518 Aug 30  2015 .bashrc*
-r-x------ 1 level02 level02  675 Apr  3  2012 .profile*
----r--r-- 1 flag02  level02 8302 Aug 30  2015 level02.pcap
```

A new file type appears: `.pcap`.
This is a Package Capture file, a file format used to store raw network packet data.
A very useful tool for inspecting network traffic is `WireShark` so we will be using it to complete this exercise.

After launching WireShark we are greeted with a nice interface showing all the sent and received packets between a server and its user.
One of the first few packets contains:
```
Linux 2.6.38-8-generic-pae (::ffff:10.1.1.2) (pts/10) 

· wwwbugs login: 
```

Continuing a little further we see a packet containing:
```
Password: 
```

With one of the final packets reading:
```
Login incorrect 
wwwbugs login: 
```

This gives us a great clue on to what is being sent in the packets between.
Digging a little deeper we can see that there are several packets that are all 1 byte long.

If we take a look at all the hex values of those bytes we get the following list of numbers:
```
66 74 5F 77 61 6E 64 72 7F 7F 7F 4E 44 52 65 6C 7F 4C 30 4C 0D
```

As an attempt to decode it we used Python:
```python
bytes.fromhex("66 74 5F 77 61 6E 64 72 7F 7F 7F 4E 44 52 65 6C 7F 4C 30 4C 0D").decode("utf-8")
```
This resulted in an output of: `'ft_wandr\x7f\x7f\x7fNDRel\x7fL0L\r`

Some strange escaped characters appear, `\x7f` and `\r`.
As you might already be familiar with `\r` a `carriage return` is sent at the end. The `\7xf` Character represent the `DEL` key, meaning the user removed some of the previously sent bytes.

Applying the `DEL` keys and removing the `carriage return` at the end we are left with the password to the flag02 user: `ft_waNDReL0L`

To finally get the flag:
```bash
su flag02
Password: ft_waNDReL0L
Getflag
Check flag.Here is your token : kooda2puivaav1idi4f57q8iq
```
