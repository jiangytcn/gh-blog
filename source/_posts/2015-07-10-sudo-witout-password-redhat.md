---
layout: post
category : Sysadmin
tagline: "Github Pages Introduction"
tags : [Sysadmin]
---

This introduction will outline specifically  how to run sudo without password under redhat OS.

1. First you need to check what privileges you have now, run command as belows
  

 

[yitao@yitao-dev bumblebee]$ ***sudo -l***

Matching Defaults entries for yitao on this host:
    requiretty, !visiblepw, always_set_home, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR LS_COLORS", env_keep+="MAIL
    PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

Runas and Command-specific defaults for yitao:
    Defaults>root !requiretty

User yitao may run the following commands on this host:
    (ALL) ALL
    (root) NOPASSWD: /usr/bin/usb-luks-encrypt
    (root) NOPASSWD: /usr/bin/imagewriter

2.Add the privilege, run the command as follows

[yitao@yitao-dev bumblebee]$ sudo visudo

   *INPUT YOUR ROOT PASSWORD TO ACCESS THE VISUDO FILE*
   Then add a record to the last line, to be specific, after the last line which says

	#includedir /etc/sudoers.d 
	
	...
	
	Defaults>root !requiretty # Line required by usb usb-luks-encrypt
	ALL ALL=(root) NOPASSWD: /usr/bin/usb-luks-encrypt
	ALL ALL=(root) NOPASSWD: /usr/bin/imagewriter
	***YOUR_NAME_HERE*** ALL=(ALL) NOPASSWD: ALL

exit.

Now you can sudo command without password.


> Written with [StackEdit](https://stackedit.io/).