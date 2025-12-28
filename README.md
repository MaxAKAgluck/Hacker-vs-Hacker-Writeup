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

But we need easier and faster access so a one-liner reverse shell is in order:

