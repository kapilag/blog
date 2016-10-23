---
title: Http/Https call with Device using Arp poisoning
date: 2016-10-23 13:19:35
tags:
- testing
- security
---

If you are still not convinced of using Emu/Simulator for monitoring HTTP/S traffic then probably this post will help in debugging network calls from mobile device.

It will save you from the pain of manually configuring proxy on mobile device and then leave it on for another person to figure out after ~30 min reason of app not working.

We will be using Arpspoofing techinque to route all the network traffic from victims(mobile) to our machine and then use port forwarding with mitproxy to monitor traffic.

*High level steps for configuring above setup is:*
1. Arp poisoning using any tool(ettercap)
1. Configure port forwarding and network rules
1. Setting up mitmproxy in transparent mode.

*Tool used*
* Ettercap
* Wireshark
* mitmproxy
*OS: OSX el Capitan*

# [Arp poisoning/spoofing](https://en.wikipedia.org/wiki/ARP_spoofing)
Arp poisoning/spoofing is a technique where victim machine is fooled to send all the data to attacker machine by making victim believes gateway is attacker machine and gateway to believe attacker machine is victim.

There are many tools available to perform this type of attack like arpspoof, ettercap, caine, nemesis etc, But i would be using [ettercap](https://ettercap.github.io/ettercap/) as part of this post.
There are n number of post available to use ettercap, so would not go into details but quick command to start ettercap is
Cursor Mode: `sudo ettercap -C`
Graphic Mode: `sudo ettercap -G`

Typical steps in above modes are:
1. Scanning host
1. Add host to targets
1. Start arpoisoning with sniffing option on.

Other way to start ettercap is in *text only -T* mode and following command seems to do the trick.  
`sudo ettercap -T -w dump -M arp /192.168.0.101/ /192.168.0.1/ output:`
some examples listed [here](https://linux.die.net/man/8/ettercap)

Once we have ettercap up & running we can now move on to Port forwarding.
 Meanwhile we can now sniff packets using Wireshark.

__TIP__: If you want to know how exactly arpspoofing work then use __nemesis__ tool instead of ettercap.

## [Port forwarding](https://en.wikipedia.org/wiki/Port_forwarding)
With arp poisoning we have routed the victim data to our machine but as we are debugging http calls we are interested on in TCP. Wireshark is too verbose for such requirement + we cannot edit & replay responses.

All the traffic on port 80(http) and port 443 needs to be transferred to port where we will be running mitmproxy(8080).

Again for port forwarding there are many tutorials but will list few mac command which might be helpful in setting up.
In order to enable port forwarding in OSX use.

`sudo sysctl -w net.inet.ip.forwarding=1`  

MAC uses *pfctl* for configuring rules for port forwarding.
Best way to configure rule is to create a new file with our configuration.   
`sudo touch /etc/pf.anchors/my_config`  

Now lets add rules to the config file.  
`rdr on en0 inet proto tcp from any to any port 80 -> 127.0.0.1 port 8080`  
`rdr on en0 inet proto tcp from any to any port 443 -> 127.0.0.1 port 8080`

Check if file is valid with `sudo pfctl -vnf /etc/pf.anchors/my_config`  
Once this is done add entry `load anchor "my_config" from "/etc/pf.anchors/my_config"` to bottom of the pf.conf file
`sudo vim /etc/pf.conf`

Once above steps are done, disable and enable pf using  
Disable: `sudo pfctl -d`  
Enable: `sudo pfctl -ef /etc/pf.conf`

__PS__: In order for the new rules to work pf needs to be disabled first on my system , took around 1hr to figure this out :(, hence please do this if something not working as expected.

Once above setup is configure all the incoming traffic to port 80/443 will be redirected to port 8080.


## [Setup of mitmproxy in transparent mode](http://docs.mitmproxy.org/en/stable/transparent.html)
I thought this step was optional as was running mitmproxy earlier but again for this as well it took me around 1 hour to figure out that mitmproxy needs to run in transparent mode in order to work with arpspoofing.

Please follow step by step tutorial over [here](http://docs.mitmproxy.org/en/stable/transparent.html) for platform specific.
Command to run mitmproxy in transparent mode.
`mitmproxy -T --host`

Once above setup is done you can see all the http/s traffic of your victim(in our case mobile phone) displayed in mimtproxy console.

Sweet isnt it ?
Do let me know your thoughts in comment section.

*Some links:*  
https://www.youtube.com/watch?v=yLbKhwnc0HY  
https://www.wireshark.org/  
https://en.wikipedia.org/wiki/Address_Resolution_Protocol  
https://en.wikipedia.org/wiki/ARP_spoofing
