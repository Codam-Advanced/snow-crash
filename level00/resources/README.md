No hints were found in this level

In order to obtain the `flag00` token. we had to find all files created by the user `flag00`
with the command:
``` bash
find / -user flag00 2>/dev/null
```

This results in:
`/usr/sbin/john` and
`/rofs/usr/sbin/john`

It shows that the user created a file `john` in the sbin folder

Running the command:
```bash
ls -l /usr/sbin/john
```
Results in: `----r--r-- 1 flag00 flag00 15 Mar  5  2016 /usr/sbin/john`

Here we can see we have read access where we find the string `cdiiddwpgswtgt` inside

Decoding this string from a caesar cipher with a shift of 22 result into the string `nottoohardhere`.

we are now able to switch to the flag00 user with the command

```bash
su flag00
Password: nottoohardhere
```

Once switched we run the command:
```bash
getflag
```
Which results in: `x24ti5gi3x0ol2eh4esiuxias`







