---
layout: post
category : linux
tagline: ""
tags : [Sysadmin]
---

This introduction will outline specifically  how to disable and enable IPV6 in RHEL 6.X

## Overview
The following commands run on redhat 6.5

Problem:
Need to disable the IPV6 in RedHat Enterprise Linux

Solution:

Disable IPV6 in RHEL:

1)      Add the below line in the modprobe.conf file

            # vi /etc/modprobe.conf

            alias net-pf-10 off
            alias ipv6 off                    # Add this for RHEL before 5.4
            options ipv6 disable=1    # Add this for 5.4 and later

Note : If any entry available like “alias net-pf-10 ipv6”, delete that entry.

2)      Also add the below entry in the network file to prevent error

            # vi /etc/sysconfig/network

            NETWORKING_IPV6=no

3)      Reboot the server to disable the IPV6

      # shutdown -r now

Enable IPV6 in RHEL:

4)      Add the below entry in the modprobe.conf file

      # vi /etc/modprobe.conf

      alias net-pf-10 ipv6

Note : Remove if any entry like “alias net-pf-10 off or alias ipv6 off or options ipv6 disable=1” in the modprobe.conf file.

5)      Change the “ NETWORKING_IPV6” parameter value to yes in the netwok file.

      # vi /etc/sysconfig/network

      NETWORKING_IPV6=yes

6)      Restart the server to enable the IPV6 support

            # shutdown -r now
Thank you.