## Hints
After having completed level00 you are left with a hint: the name `John`.
This will become of use later in this exercise.

## Solution
Whilst looking for clues in level00 we found a peculiar thing after reading the `/etc/passwd` file: the flag01 user did not have its hashed password hidden in the /etc/shadow file!
Instead it reads: `42hDRfypTqqnw`.

Doing a bit more research online we found that there are several password-cracking tools out there, one being named [`John the Ripper`](https://github.com/openwall/john).
Having seen the name `John` in the previous exercise it was the logical thing to try out next.

So after saving the hash inside of a file:
```bash
$ echo "42hDRfypTqqnw" > HashedFlagLevel01
```

And running john on it:
```bash
$ john HashedFlagLevel01 --show
?:abcdefg

1 password hash cracked, 0 left
```

We are left with the password to the flag01 user! `abcdefg`

All that is left is logging into the user and retrieving the flag.
```bash
$ su flag01
Password: abcdefg

$ getflag
Check flag.Here is your token : f2av5il02puano7naaf6adaaf
```