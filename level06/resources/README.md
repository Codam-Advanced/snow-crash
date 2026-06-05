## level 06

Having completed level 5 we can start our journey with level 06!
As always we start with a simple `ls` to see what we're dealing with: 

```bash
level06@SnowCrash:~$ ll
total 24
dr-xr-x---+ 1 level06 level06  140 Mar  5  2016 ./
d--x--x--x  1 root    users    340 Aug 30  2015 ../
-r-x------  1 level06 level06  220 Apr  3  2012 .bash_logout*
-r-x------  1 level06 level06 3518 Aug 30  2015 .bashrc*
-r-x------  1 level06 level06  675 Apr  3  2012 .profile*
-rwsr-x---+ 1 flag06  level06 7503 Aug 30  2015 level06*
-rwxr-x---  1 flag06  level06  356 Mar  5  2016 level06.php*
```

We are greeted with a `level06` binary and a `level06.php` script.
Running Ghidra on the binary we quickly found out that its another wrapper to simple elevate SUID permissions as seen in previous exercises, internally calling the PHP script.
So next up we're going to take a look at the php file: 


```bash
level06@SnowCrash:~$ cat level06.php 
#!/usr/bin/php
<?php
function y($m) { $m = preg_replace("/\./", " x ", $m); $m = preg_replace("/@/", " y", $m); return $m; }
function x($y, $z) { $a = file_get_contents($y); $a = preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a); $a = preg_replace("/\[/", "(", $a); $a = preg_replace("/\]/", ")", $a); return $a; }
$r = x($argv[1], $argv[2]); print $r;
?>
```
This output is really confusing to work with so to make our lives a bit easier we format the file ourselves:

```php
#!/usr/bin/php
<?php
function y($m) {
        $m = preg_replace("/\./", " x ", $m);
        $m = preg_replace("/@/", " y", $m);
        return $m;
}
function x($y, $z) {
        $a = file_get_contents($y);
        $a = preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a);
        $a = preg_replace("/\[/", "(", $a);
        $a = preg_replace("/\]/", ")", $a);
        return $a;
}

$r = x($argv[1], $argv[2]);
print $r;
?>
```

Now we see the challenge and for this level and our worst nightmare ***regex...***

## Solution

Being swarmed with unreadable regex is never fun so we start with dissecting it line by line.
We quickly found out how to exploit this, there is a special character found within the `function x`.
In the first pattern we find a `/e`, this is a modifier that can evaluate PHP expressions during runtime.
This functionality is deprecated as of PHP 5.5, and removed in 7.0.
Luckily for us the version running on SnowCrash is 5.3.10!

The way the script works is it reads `$argv[1]` as a file name and retrieves the contents using `file_get_contents`.
That content is then used by the following lines of `preg_replace`, changing the output along the way. 
Within this exercise there were quite a few red herrings and things that weren't of use to get our flag.
Since the entire `function y`, the last two `preg_replace` calls and `$argv[2]` did not contribute to the solution we will not go into detail about them.

That leaves us with the `preg_replace` call that contains the `\e` modifier.
What the `\e` modifier does is call `eval()` on its contents, which in turn escapes all quotes.
Having all the quotes escaped meant that our first approach of trying to use quotes to break out of the double quotes inside the replacement argument and insert our own function call did not work.

So whats left? Well we can create a match for the pattern containing the `/e` by starting it with a `[x` and ending with a `]`.
Next up we are able to insert some code and have it be executed because of a cool PHP feature called `Complex Syntax`.
Our goal is to call `shell_exec` with `getflag` as its argument.

We are going to cheat a little here, our plan is to use `Complex Syntax` to treat the output of `shell_exec` as a variable, which will be undefined and throw an error, giving us our token right away!

`Complex Syntax` works as follows, you can wrap expressions inside of a `{}` to have them be evaluated.
Then inside of that we use the `${}` syntax, which causes a variable variable lookup, executing our `shell_exec` first to determine the variable name.

Since `shell_exec` expects a string we can cheat a little more and add `getflag` into our input without surrounding it in quotes.
PHP will assume it to be a string, only giving us a notice of the error.

Adding all of this together we finally get the input `[x {${shell_exec(getflag)}}]`, adding it to a file:

```bash
level06@SnowCrash:~$ echo '[x {${shell_exec(getflag)}}]' > FlagGrabber.txt
```

And passing it to the executable we are relieved from our regex journey:

```bash
level06@SnowCrash:~$ ./level06 FlagGrabber.txt
PHP Notice:  Use of undefined constant getflag - assumed 'getflag' in /home/user/level06/level06.php(4) : regexp code on line 1
PHP Notice:  Undefined variable: Check flag.Here is your token : wiok45aaoguiboiki2tuin6ub
 in /home/user/level06/level06.php(4) : regexp code on line 1
```

Filtering through the error messages we can find our flag: `wiok45aaoguiboiki2tuin6ub`!
