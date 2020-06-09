---
layout:     post
title:      CentOS 7 first steps!
date:       2020-06-09 09:44:54
author:     yFStein
summary:    First steps to do after installing a fresh CentOS7 cloud/vm server
categories: CentOS7
thumbnail:  heart
tags:
 - centos7
 - initial
 - adduser
 - ssh
 - firewall
---

Everyone today want to start their own server. So their are very much servers which are completely insecure and did not have any well configuration. In front of that, the servers will be hijacked to work as bots which we know as daily pain.

In this tutorial i want to show you today which first steps after creating a cloud or vps server with CentOS 7 i would done, to get the first things secured.


## Creating sudo/wheel user

To not use the root user, we will create a new user which we will add to the wheel group. The wheel group is a usergroup of additional users with root-permissions. The focus is to not use the root user as default user for daily business.

So first, we will create the user - in our tutorials he will be named as "monty".

`adduser monty`


Now the user should have a password, to get a working authentication:

`passwd monty`


Just type in 2 times the password for the user. After that, its just a normal user. So we want, that this user has rights to use "sudo" to use commands with root privileges:

`usermod -aG wheel monty`


So the user is created, has a password and has "sudo" rights. But in further, we want to login to this user not with the created password - we want to use public-key authentication.

So we need to create the file for the user, where we can save the public-key, which we will open with our private key.

You dont know what public-key authentication is? Then i think you should [start over here](https://yfstein.github.io/create-and-understand-private-and-public-keys/) to learn what it is and how to create a key-pair.

So lets create the ssh folder to the monty user:

`mkdir /home/monty/.ssh`


After that, we need to create the Authorized_Keys File to load the public keys, which can be used to login the user. Be aware - we are using "nano" as texteditor. If its not installed on your system you can use another editor or install it with `yum install -y nano`. Lets create the public-key-file:

`nano /home/monty/.ssh/authorized_keys`


Fill in here the "ssh-rsa" key which [we have created in this tutorial](https://yfstein.github.io/create-and-understand-private-and-public-keys/).

Now save and exit the file and we can start to change the SSH configuration to login with monty and his freshly created public-key login.


## Configuring the SSH Daemon

Because their are a lot of bots out their, that scanning the internet, looking for open ports to connect and so on we will change some settings to the SSH Daemon, so its would be harder for the bots to bruteforce your passwords or connecting to the default port.

Just open the SSH config file: `nano /etc/ssh/sshd_config` and edit the following lines:

change to a valid port above 1024 and remove the hash - in this case we change to 5876:

Find: `#Port 22`

Replace with: `Port 5876`

So the default port scanner didnt reach your SSH Port 22 and eventually giving up for the first. That not means that nobody can do additional port scan to find your SSH Port. But basically its absolutely recommended to change the SSH port.

So we have created the additional user "monty" so we dont need to use the root user to login directly over SSH. So we can change the PermitRootLogin to no.

Find: `#PermitRootLogin yes`

Replace with: `PermitRootLogin no`


So now coming the part with the created AuthorizedKeysFile - here we will check if the line looks exactly like the following one:

Find and Check: `AuthorizedKeysFile .ssh/authorized_keys`

So the SSH Daemon can check everyones home folder for the directory .ssh with the authorized_keys file, where the public key for every user should be stored. Surely, you can change to every another folder but its a default-recommended way.


Because we want to login in the further with our public-key and not with a user-password we will deactivate the PasswordAuthentication:

Find: `#PasswordAuthentication yes`

Replace with: `PasswordAuthentication no`

So now we coming to a point, where i am self everytime struggling. Should i turn it on, or should i turn it off. But in the last years i got very excited over turning the ChallenegeResponseAuthentication off. But why? So, for the first, i had some servers with a good income of traffic and ChallenegeResponseAuthentication turned off since years. We never had any corruption, "hacks" or anything like this. After a long internal discussion we decided to create some Honeypots and share the IPs where ChallenegeResponseAuthentication is turned on. After just one week their we noted the first brake-in to the system. Sure, the honeypot wasnt secured well, but for testing purposed more than enough - so we decided for the first to still no turn on the ChallenegeResponseAuthentication.

check that this line looks like this, mostly its already does:
Find and Check: `ChallenegeResponseAuthentication no`

So because we are here on command line and not graphical ui, we also dont need a function that anybody can connect over X11. So we will find and replace:

Find: `#X11Forwarding yes`

Replace with: `X11Forwarding no`


Now save and exit the file. After we created the changes to the sshd_config file, we need to restart the SSH Daemon:

`systemctl restart sshd`


After that we should be able to login as monty with our private key over ssh port 5876.


## Update the system

Because not every system which we are creating on our providers infrastructure is up-to-date we will update the repositorys, check for updates and install them:

`yum update -y`


## Installing firewalld

To get a bit more secure, we will use firewalld as firewall for our systems. With the following command we will install it:

`yum install -y firewalld`


## Configuring firewalld

Before we can enable and start the firewalld, we need to deactivate a deprecated and insecure option:

`nano /etc/firewalld/firewalld.conf`


Please search for "AllowZoneDrifting" and change the yes to no and remove the hash:

`#AllowZoneDrifting = yes`

`AllowZoneDrifting = no`

Older versions of firewalld had undocumented behavior known as "zone drifting". This allowed packets to ingress multiple zones - this is a violation of zone based firewalls. However, some users rely on this behavior to have a "catch-all" zone, e.g. the default zone. You can enable this if you desire such behavior. It's should be disabled for security reasons.

After we done that we can enable and start the firewall:

`systemctl enable firewalld`

`systemctl start firewalld`


But because we have a changed SSH port, we need to add the new port to the firewall and remove the old one:

`firewall-cmd --zone=public --permanent --service=ssh --add-port=5876/tcp`

`firewall-cmd --zone=public --permanent --service=ssh --remove-port=22/tcp`


Now we just need to reload the firewall to apply the changes:

`firewall-cmd --reload`


And the firewall should block any traffic coming to port 22, because we have close it and the SSH connection should still possible to port 5876.


## Conclusion

In this tutorial we learned to take just a few hardening so that our new cloud/vm server wont get hijacked within the first few days.


## Questions?

[Open an Issue](https://github.com/yfstein/yfstein.github.io/issues/new) and let's chat!


## Credits

- [yFStein](https://github.com/yfstein) - Me, for investing the time to build up this crap
- [DevFalko](https://github.com/devfalko) - Thanks for all the support regarding to the blog articles and setup testing.


## Contributing

Issues and Pull Requests are greatly appreciated. If you've never contributed to an open source project before I'm more than happy to walk you through how to create a pull request.

If you want your own blog post for my blog so just fork the repo, write the post and send me a pull request. Or did you have questions or issues with my tutorials? Just send a issue and we can chat about it!


##### License: MIT

<!---
Contributors's Certificate of Origin
By making a contribution to this project, I certify that:
(a) The contribution was created in whole or in part by me and I have
    the right to submit it under the license indicated in the file; or
(b) The contribution is based upon previous work that, to the best of my
    knowledge, is covered under an appropriate license and I have the
    right under that license to submit that work with modifications,
    whether created in whole or in part by me, under the same license
    (unless I am permitted to submit under a different license), as
    indicated in the file; or
(c) The contribution was provided directly to me by some other person
    who certified (a), (b) or (c) and I have not modified it.
(d) I understand and agree that this project and the contribution are
    public and that a record of the contribution (including all personal
    information I submit with it, including my sign-off) is maintained
    indefinitely and may be redistributed consistent with this project
    or the license(s) involved.
Signed-off-by: [yFStein info@meikelbloch.de]
-->
