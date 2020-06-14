---
layout:     post
title:      CentOS 7 SSH private/public key not accepted
date:       2020-06-15 023:51:36
author:     yFStein
summary:    Some basically troubleshooting to fix a not accepted SSH private/public key
categories: CentOS7
thumbnail:  heart
tags:
 - centos7
 - public
 - private
 - ssh
 - firewall
---

In the last days we got a little problem after setup a CentOS 7 server, which seems to be new. But after fast troubleshooting we got the issue.


## Check the private key

For the first we would take a look at our private key, which we are providing from our workplace to login with the SSH server. You should know where you have it placed to. In our case we had it recommended under the following path:

`~/.ssh/id_rsa`

So we will check if their is any BOM type or misspelling. Just to check it further, copy your key again to get safe that its the right one.

`sudo nano ~/.ssh/id_rsa`

If you have a Windows System it should be also placed under `C:\Users\YourUserName\.ssh\id_rsa`

You can also define in the `C:\Users\YourUserName\.ssh\config` file or under linux in `~/.ssh/config` something like this:

```
Host your-hostname
        HostName your-host-or-ip
		Port your-ssh-port
		IdentityFile ~/.ssh/id_rsa
```

With that you will create a host template for "your-hostname" with the specific "HostName" which contains the IP adress or the reachable hostname. On the point "Port" you should enter your SSH Port. The "IdentityFile" should be the location for the your private key. You can use like below for Windows and linux if its on the Users Home Folder in the ssh folder with the name "id_rsa".


## Check the public key

To check the public key we will check the file for eventually misspelling or BOM typings. Just open the file located on your server:

```nano ~/.ssh/authorized_keys```

To make sure just replace the key and check that their is no second line or any BOM character.


## Check the rights of the public key

To set the correct rights for the key an folder use the following commands:

```
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh/
```


## Conclusion

In our case we got the issue fixed after repairing the rights for the public key and the folder. Which fixed your your problem or did you had another solution?


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
