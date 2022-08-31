---
title: Portspoof Bypass with a simple trick
author: Zeyad Azima
categories: SecuritySolutions
tags: ["Portspoof","Nmap","Scanning","Ports","Pentest","Security"]
match: true
---


<img src="assets/img/Portspoof/PORTSPOOF.png">

# What is Portspoof ?

Portspoof is meant to be a lightweight, fast, portable and secure addition to the any firewall system or security system. The general goal of the program is to make the reconessaince phase slow and bothersome for your attackers as much it is only possible. This is quite a change to the standard 5s Nmap scan, that will give a full view of your systems running services. [Read More.](https://drk1wi.github.io/portspoof/)

# Install Portspoof

Note: Don't forget to run all commands as sudo in new versions of linux. So, you avoid problems
You can install portspoof from the following blog.

or

From the following commands:
```
git clone https://github.com/drk1wi/portspoof.git
```
```
sudo ./configure && make && sudo make install
```
```
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp -mtcp --dport 1:65535 -j REDIRECT --to-ports 4444
```
Copy the following bash script and save it into set-portspoof.sh and run it as sudo:
```
#! /bin/bash

spoofPorts="1:19 23:24 26:52 54:79 81:109 112:122 124:442 444:464 466:586 588:891 893:2048 2050:8079 8081:32800 32801:65535"

for prange in ${spoofPorts}; do

iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport ${prange} -j REDIRECT --to-ports 4444

done
```

# Run Portspoof

Now lets run portspoof. But, before we run it the tools directory contains the config file and signatures file:

```
sudo portspoof -c /tools/portspoof.conf -s /tools/portspoof_signatures -D
```
Now we have Portspoof running lets start the party

# How Portspoof is working ?

Portspoof got a huge number of signatures when you connect to any port as we configured before in iptables it will forward all connections to portspoof services and will response with [syn-ack] that the ports are open. Then we won't be able to identify the real services. Also, portspoof is responsing with banners etc. So, its hard to identify the actual running real services. But, portspoof is not affecting the actual real running services ( as we configured in the bash script ).
Portspoof working steps:

```
1- Attacker perform port scanning using nmap 
```
```
2- The victim machine look at the iptables rules and forward the connection to portspoof services 
```
```
3- portspoof response with [syn-ack] 
```
```
4- Nmap identify ports as open and running 
```
```
5- attacker try to enumerate version of protocols & banners 
```
```
6- portspoof response based on the config & signatures (wich is fake) 
```
```
7- Nmap output based on the portspoof response
```

# Bypassing Portspoof
We need to know how portspoof is dealing with our connections. So, we be able to generate a bypass scenario. 
Lets try to connect manaual using netcat or telnet

I have used netcat but the connection were end without any output for a reason

netcat:

<img src="assets/img/Portspoof/Screenshot 2021-11-14 132154.png">

I tried telnet after that and the connection is closed by the host (victim):

<img src="assets/img/Portspoof/Screenshot 2021-11-14 132556.png">


As we can guess it seems that the portspoof just creating the banners, etc and close the connection.
The victim got port 21 (ftp) open by default so lets try to connect it.

<img src="assets/img/Portspoof/Screenshot 2021-11-14 133148.png">

As you can see the real services didn't close the connection. Then for sure the fake one close the connection. 
Steps to identify the real running services:

```
1- Connect to the service 
```
```
2- if we recive data and services closed connection then its fake 
```
```
3- if we recived data and send any command and recived another data then its real service
or the connection didn't closed 
```
```
4- Get the real service
```

# Automate the process

 We gonna automate the process using python and we gonna try it on FTP services
 But, First lets try to scan with nmap.
 
<img src="assets/img/Portspoof/Screenshot 2021-11-14 134109.png">

As you can see its a miss and most of the ports identified as open and few as closed.
So, we gonna use the following script to test it.

Script:

<img src="assets/img/Portspoof/Screenshot 2021-11-14 135046.png">

To avoid false positives we gonna use the commands for each protocol and we gonna try with ftp & http.
The script is creating a connection with ftp services first the recive response then try to login with anonymous and recive the second response and save it in response variable. After that is comparing if the response contains the Login word cause when we try to login in FTP is printing Login word in each fail and success try.
If the Login is exist it will identify the services as open else is gonna just pass it.

The same for the HTTP services but its checking if the html word is exist in the response.

Now let's run it:

<img src="assets/img/Portspoof/Screenshot 2021-11-14 135558.png">

We got the ftp port as opened.
Now you know how to bypass it you can write your tool on your own way to identify the services that you want or to identify if the connection is alive or no.

Video:

[![Watch the video](https://i9.ytimg.com/vi_webp/J2umlrFNuMk/mqdefault.webp?sqp=CPjmvJgG&amp;rs=AOn4CLB_P82c5CzxMBjfciFAchB8BKKcrg)]([https://youtu.be/T-D1KVIuvjA](https://youtu.be/J2umlrFNuMk))
