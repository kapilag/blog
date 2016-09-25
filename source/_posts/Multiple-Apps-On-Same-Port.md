---
title: Multiple Apps On Same Port
date: 2016-09-25 20:15:08
tags:
 - networking
 - juniordevforlife
---

Few day back one of my collegue asked me very basic question, can we run two application on same port.
The answer to this question is so obvious for most of us but actual reasoning why we cannot run two application was somehow missing for me. This led me to search lots of blogs, wikipedia, Stackoverflow, Networking concept and finally whenever everything looks sorted some or other thing came up.

Before answering this question let refresh some of the concepts(taken from wikipedia)
### What is a Port
In IP protocol suite , port is end point of communication in operating system. It is a logical construct that identifies specific processes. Port number is a 16-bit unsigned integer ranging from 0 to 65535.

### What different Protocols are for interest over here
UDP and TCP

### What is Socket
A network socket is an endpoint of a connection across a computer network. Socket is identified by {SRC-IP, SRC-PORT, DEST-IP, DEST-PORT, PROTOCOL}.
Then there is socket API which is provided by OS to bind sockets over ports.

### What is "Address already in use Error"
When application try to use same port which in bound to some other socket then OS will inform via relevant error code that address was already in use.

Now back to main question :"Can we use same port for listening?"
Functionality of not allowing re-use of same port as we seen above is implemented by Kernel, Reasoning behind such implementation is that if client send packets over network on certain port, then kernel needs to know which application to point to which can serve the request.

But as you see again from socket question we can run two different TCP and UDP application on same port, the reason for this is in all most all OS implementation port ranges are different for UDP and TCP , that mean 0 -65535 for tcp and 0-65535 for UDP. And every IP connection is defined by socket tuple above which will be unique in this case[DNS uses port 53 for both TCP and UDP]

One another way of running application on same port is installing different network card or in other words running on different interfaces , which will have different IP addresses assigned to it.(see Socket question again)

Having said all the above we can conclude that we cannot run two tcp application on same port as Kernel(OS) will not allow this.
But Hold on

I ran two application one Java and other Ruby on the same port and below is the screen shot of the output.
{% asset_img MultipleAppsSamePort.png multipleapps %}

Now what the heck is this? , all the conclusion is nullified seeing the above output. Both Java and Ruby are listening on the same port ?😵

However if we give a close look to the screenshot there is IP difference. Webrick/Sinatara is listening on IPv4 range and Jetty on IPv6 range.
Hence we are able to have to application bound on same port , there are other explanation also to this using PORT REUSE in socket programming.

All the above information is based on the reading of some random blogs and stack overflow, This was a rabbit hole for me and above explanation might go wrong in very technical sense but I tried to make it simple and for sure there are some glitches in explanation.
Please correct me if something is ambiguous or wrong in it in comments section.

Till then
> Stay hungry Stay foolish!.
