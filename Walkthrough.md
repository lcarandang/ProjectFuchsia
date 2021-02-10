This is a red team walkthrough for Project Fuchsia. It is a boot2root challenge in which our goal is to reach /root and read the flag to complete the challenge.

You can download it from here: https://bit.ly/36SJ9Zh

![0- VM login](https://user-images.githubusercontent.com/69183431/107474925-22b3ba80-6b28-11eb-8626-5bb10f8cd5c9.png)

# Penetration Methodologies #

## Scanning ##
* Netdiscover
* Nmap
* Nikto

## Enumeration ##
* Enum4linux
* Dirb
* WPscan
* Browsing port 80

## Exploiting ##
* Login to Samba from Wordpress credential posts
* Login to SSH with basic user

## Privilege Escalation ##
* Copy /etc/passwd and /etc/shadow
* John the password


We'll begin with network scanning to identify the IP of the VM with the help of netdiscover.

![1- netdiscover](https://user-images.githubusercontent.com/69183431/107474957-319a6d00-6b28-11eb-9f5f-b40e7eb19e00.png)

Our target IP is 192.168.56.112. Now, let's scan the services and ports via nmap

![2- nmap](https://user-images.githubusercontent.com/69183431/107474975-3b23d500-6b28-11eb-9428-751b0e7557eb.png)

We obtained the results from the scan and notice there are quite a few open ports. We can see that the machine is running FTP, HTTP, SAMBA, and a couple other closed ports. We start up a nikto scan before heading to our browser.

![3- nikto](https://user-images.githubusercontent.com/69183431/107474996-42e37980-6b28-11eb-948e-83d0501a3153.png)

Our scan results are pretty short but we do get a helpful clue that the machine has the myphpadmin plugin installed. That means that our target computer is running a LAMP server. Next we begin enumerating the VM with enum4linux.

![4- enum4linux pt1](https://user-images.githubusercontent.com/69183431/107475010-4aa31e00-6b28-11eb-8a4d-9fb97bc91fa6.png)

With it, we were able to see the samba drive names and a couple usernames to keep note.

![5- enum4linux pt2](https://user-images.githubusercontent.com/69183431/107475044-5abafd80-6b28-11eb-80ee-49a68237e9e0.png)
![6- enum4linux pt3](https://user-images.githubusercontent.com/69183431/107475070-64446580-6b28-11eb-8fd2-bc3f8dfec3fb.png)

Attempting to connect to the share drives anonymously fails.

![6 5- Files Other smb](https://user-images.githubusercontent.com/69183431/107475082-6a3a4680-6b28-11eb-8e20-0dd5f9ad30c6.png)

Next up we'll finally hit up port 80. We arrive at a default apache page. Before we assume this is all there is, we should perform a dirb scan.

![7- apache default](https://user-images.githubusercontent.com/69183431/107475133-7e7e4380-6b28-11eb-9975-c7706b8626a1.png)
![8- dirb](https://user-images.githubusercontent.com/69183431/107475151-85a55180-6b28-11eb-94b6-964617179bc9.png)

We find that there is a wordpress site being hosted from the VM, and before we go back to our browser, I throw up a WPscan to look for any theme or plugin vulnerabilities. It shows us that Profile Builder and WD Google Maps are out of date, something to possibly look into later. For now, we go back to Firefox and check out the wordpress.

![9- wordpress pt1](https://user-images.githubusercontent.com/69183431/107475168-8d64f600-6b28-11eb-8bc7-126a0914abc6.png)

It looks to be a Pokemon-themed blog with several posts. Skimming each of them reveals they are mainly text-based posts, much more so than the homepage. It seems the next clue is written between the lines.

After carefully examining each of them, I noticed that two posts contained similar names. One of them even speaks about finally obtaining a login which definitely sounds like a hint.

![10- wordpress pt2](https://user-images.githubusercontent.com/69183431/107475186-95249a80-6b28-11eb-8848-934b06c7bfbd.png)

Comparing that to some of the text from the other post...

![11- wordpress pt4](https://user-images.githubusercontent.com/69183431/107475202-9bb31200-6b28-11eb-9c96-8c56c42e9f08.png)
![12-wordpress pt3](https://user-images.githubusercontent.com/69183431/107475210-a1a8f300-6b28-11eb-805e-d2d95cfbcdcb.png)

...and it looks like we have a set of credentials to try out. We go back to Files > Other > smb://192.168.56.112, and we're in!

![13- smb pt1](https://user-images.githubusercontent.com/69183431/107475224-a79ed400-6b28-11eb-88f2-41696301efa5.png)
![14- smb pt2](https://user-images.githubusercontent.com/69183431/107475237-abcaf180-6b28-11eb-8aa7-17e5bc8b0ff1.png)

It looks like a bunch of game files and folders. There's probably a hidden flag here. It's best we turn on the option to reveal them.

![15- smb pt3](https://user-images.githubusercontent.com/69183431/107475253-b08fa580-6b28-11eb-8f46-0a88d4f2844e.png)

Looking inside the TXT_files_Characters directory disclosed a hidden .credentials.txt file. Opening it gave us SSH creds for a basic user, Koga, while also hinting to the fact there is a knock sequence blocking port 22.

![16- smb pt4](https://user-images.githubusercontent.com/69183431/107475268-b5545980-6b28-11eb-9492-c0dfc1f323f8.png)
![17- smb pt5](https://user-images.githubusercontent.com/69183431/107475289-ba190d80-6b28-11eb-9d13-f240a08be20d.png)

We go back to checking the other files and folders in the samba drive, and we eventually find something peculiar in the MP3_files_Characters_AttackSounds folder. Typically you would assume this is filled with only .mp3 files, but there is a .txt file trying to blend in with the names.

![18- smb pt6](https://user-images.githubusercontent.com/69183431/107475303-be452b00-6b28-11eb-89c0-00911989fd33.png)

Opening it reveals the port knock sequence! Besides the numbers, the text reveal that this was something they wished to conceal. 

![19- smb pt7](https://user-images.githubusercontent.com/69183431/107475319-c3a27580-6b28-11eb-9dd6-9233cad2ba7f.png)

Time to do the opposite! A quick nmap scan shows a certain port 22 open that previously wasn't there.

![20- ssh open](https://user-images.githubusercontent.com/69183431/107475334-c8672980-6b28-11eb-9146-27b77079566e.png)

Now we use the basic SSH credentials to login and take a look around. The opening message reveals how this account is restricted.

![21- koga pt1](https://user-images.githubusercontent.com/69183431/107475345-cd2bdd80-6b28-11eb-8933-f67116a6aff8.png)

Some preliminary commands show that we can't use cd nor does Tab autocomplete work to find out what commands we can and can't use. However, some more messing around leads to us knowing we can use touch and cat! That's exactly enough for what we need.

![22- koga pt2](https://user-images.githubusercontent.com/69183431/107475362-d7e67280-6b28-11eb-87f1-fb6deb797935.png)
![23- koga pt3](https://user-images.githubusercontent.com/69183431/107475364-d9179f80-6b28-11eb-99d1-f8b684c70b0b.png)

We cat the /etc/passwd and /etc/shadow files so we can copy the first line with root in both of them into text files for our local Kali. 

![24- koga pt4](https://user-images.githubusercontent.com/69183431/107475367-d9b03600-6b28-11eb-9360-9f4c509cf4ad.png)
![25- koga pt5](https://user-images.githubusercontent.com/69183431/107475368-dae16300-6b28-11eb-9c6b-c24bd4a1141b.png)

We then combine the files into one using unshadow and run John the Ripper on it.

![26- unshadow crack](https://user-images.githubusercontent.com/69183431/107475414-ef256000-6b28-11eb-9cad-f3d0c8834a5d.png)

Cracking passwords is almost always a lengthy endeavor. At this stage of the attack, you'll need patience. You never know if you get the password in 30 minutes...

![27- john pt1](https://user-images.githubusercontent.com/69183431/107475417-f0568d00-6b28-11eb-87b3-11c736e2d32a.png)

...or 4 and a half hours later.

![28- john pt2](https://user-images.githubusercontent.com/69183431/107475431-f3ea1400-6b28-11eb-91b3-dcd9df277e1c.png)

So we hope you took a break while that ran! Now let's use those root credentials. It looks like they disabled root login through SSH. Although switching users from koga works around that just fine.

![29- no root ssh](https://user-images.githubusercontent.com/69183431/107475438-f51b4100-6b28-11eb-9e0b-9f8195317087.png)
![30- su root](https://user-images.githubusercontent.com/69183431/107475447-f77d9b00-6b28-11eb-9bbd-685b33fdaeed.png)

And here's how it looks from the actual VM.

![31- VM flag](https://user-images.githubusercontent.com/69183431/107475450-f9475e80-6b28-11eb-8d01-f67f4e176b85.png)

We hope you enjoyed our machine! Like our flag says, we only highlighted the most common way to get to root. But don't let that stop you from trying to get in through any of the other doors we have revealed and left around!
