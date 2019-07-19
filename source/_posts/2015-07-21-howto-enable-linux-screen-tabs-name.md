---
layout: post
category : Sysadmin
tagline: "Enable screen tabs name "
tags : [Sysadmin]
---

This introduction will outline specifically on how to show tab name on an active screen

Linux screen is a very powerful tool for sysadmin when you need to run a long running process without kept your computer connecting to the remote server

here is the deatils introductions about [Screen](http://linux.die.net/man/1/screen)

----------

There is two ways to display tab name 

 - Configure .screenrc
 

    Add this to your .screenrc file:
    
    `caption always "%{= kw}%-w%{= BW}%n %t%{-}%+w %-= @%H - %LD %d %LM - %c"`
    
    After you restart your screen, there's a status bar below showing the current tab name, and as a bonus your current host name and time -- modify them away at will if you so wish.
    
    To rename a tab, press ctrl+a A and give it a new name.
    
 -  Operate at existing screen
	
	Sometimes you create a screen but havn't enable captions, it's not convient to switch among tabs, you can run following commands to enable screen tabs

	 - First you need to detach current screen
		 - *`ctrl + a +d`*
		 - *`screen  -S <YOUR SCREEN NAME HERE> -X caption always "%{= kw}%-w%{= BW}%n %t%{-}%+w %-= @%H - %LD %d %LM - %c"`*
		 - Attach to your screen
			 - *`screen -r <YOUR SCREEN NAME HERE>`*
	
	Now your screen will be looks like this
	
 ![screen image after enable tabname](http://7fvkn1.com1.z0.glb.clouddn.com/screen.jpg)