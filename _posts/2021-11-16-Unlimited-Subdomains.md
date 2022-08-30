---
title: Unlimited subdomain enumeration workflow
author: Zeyad Azima
categories: BugBounty
tags: ["Bugbouny","Bug bouny","Enumeration","Recon","Subdomains"]
match: true
---

<img src="/assets/img/Subenum/Subdomain-enum-workflow.png">

# Contents
- <a href="#introduction">Introducation</a>

- <a href="#tools-installation">Tools Installation</a>

- <a href="#workflow">Workflow</a>

- <a href="#conclusion">Conclusion</a>


# Introduction
In this blog, i will share my subdomain enumeration workflow in a managed and arranged way. Also, i am gonna share the way for finding some domains that maybe no one look at it before which gonna increase your chance for finding vulnerable endpoints. We gonna save all of our subdomains in a database and sort it by arranging it in the subdomain level way. 
e.x:( level 1="test.example.com", level 2="admin.test.example.com" & so on)

# Tools Installation
You can use any kind of tools that you want. Even all tools & methods.
Why?:
Cause each tool can get subdomain that the others cant get & so on. So, it will increase our subdomains scope.

List of tools i'll use:
```
subfinder 
findomain 
sublist3r
amass
assetfinder 
scilla 
db 
dlevel
```
 
Note: I'm not gonna use amass but you can use it if u want & don't forget to move any tool you download to /usr/bin directory.

- Subfinder

You can install it using apt command:
```
sudo apt install subfinder
```
or download it from the releases:

https://github.com/projectdiscovery/subfinder/releases
 
- Findomain

You can download it from the releases:

https://github.com/Findomain/Findomain/releases/tag/5.0.0
 
- Sublist3r

You can install it using apt:
```
sudo apt install sublist3r
```
or download it from github using git:
```
git clone https://github.com/aboul3la/Sublist3r.git
```
 
- amass

You can install it using apt:
```
sudo apt install amass
```
or from github:

https://github.com/OWASP/Amass/blob/master/doc/install.md

 
 - Assetfinder

You can install it from github:

https://github.com/tomnomnom/assetfinder#install

 
- Scilla

You can install it from github:

https://github.com/edoardottt/scilla#building-from-source

 
- gobuster

You can install it with apt:
```
sudo apt install gobuster
```
or download it from releases:

https://github.com/OJ/gobuster/releases/tag/v3.1.0

 
- db

First install the requirements: 
```
sudo pip3 install Flask flask_sqlalchemy loguru uuid fire csv
```
This tool created by: ihebski

and i have added a very simple little edit by adding print function to print the subdomains so you can redirect the output to a file.
 
now download the tool from github using git:
```
git clone https://github.com/Zeyad-Azima/db
```
Again don't forget to move the downloaded tools to /usr/bin directory.
 
- dlevel

You can download it from github:
```
https://github.com/MPaandeey/dlevel#install
```

# Workflow
Now let's start our workflow. I am going to use mail.ru as a target.

Starting with subfinder:
```
subfinder -d mail.ru -o subfinder.txt
```
Lets pass the domains we got with subfinder to db tool
```
cat subfinder.txt | db
```

<img src="/assets/img/Subenum/Screenshot 2021-11-16 140039.png">

as you can see subfinder got "54591" subdomains and all now in our database.

Now the next tool findomain:

```
findomain -t mail.ru -o
```

findomain is saving the ourput file with domain name and .txt (mail.ru.txt)
Let's pass it to db tool

```
cat mail.ru.txt | db 
```

<img src="/assets/img/Subenum/Screenshot 2021-11-16 140825.png">

As you can see the red colored domains db won't add it cause its a duplicated domains and just add the non-existing domains (white colored)  in  the database

<img src="/assets/img/Subenum/Screenshot 2021-11-16 141036.png">

wow 1083 new domains have been added from findomain that subfinder won't able to find.
Let's keep the process on with other tools

sublist3r results:

<img src="/assets/img/Subenum/Screenshot 2021-11-16 141414.png">

We just got 1 new domain with in sublist3r results. You can use -b option with sublist3r to bruteforce the subdomains.

assetfinder results:

<img src="/assets/img/Subenum/Screenshot 2021-11-16 142203.png">

another 608 domains from assetfinder

Scilla results:

<img src="/assets/img/Subenum/Screenshot 2021-11-16 151627.png">

I can say that scilla did a really rock job with 13278 new domains added to our database

Now we gonna to the second part and its to bruteforce subdomains using gobuster. Also, you still can use scilla or amass for bruteforcing but for me gobuster is more fast than scilla.
I'm gonna use the following wordlist for bruteforcing.

https://github.com/ZephrFish/Wordlists/blob/master/HugeDNS.7z 

gobuster command:
```
gobuster dns -d mail.ru -w EnormousDNS.txt -o gobuster.txt
```

But, since gobuster is giving me error cause domains are for the same ip and if i used wildcard its gonna be a miss. Then, we will replace gobuster with scilla

Scilla command:
```
scilla subdomain -target mail.ru -o txt -w EnormousDNS.txt
```

Scilla bruteforcing results:

Since the wordlist is enormous (over 3 million word)

I had stopped it atfter a while.

<img src="/assets/img/Subenum/Screenshot 2021-11-16 153841.png">

As a results of bruteforcing we got another 3051 new subdomains.

Now let's check the total of subdomains and save to all.txt
command:
```
db mail.ru >> all.txt
```

<img src="/assets/img/Subenum/Screenshot 2021-11-16 154153.png">

A huge differance as you can see from 54k domains to 72k domains

We gonna use dlevel to arrange the subdomains

command:
```
cat all.txt | dlevel 2 >> level-2-mail.ru.txt
```

This command gonna get the subdomains (level 2 subdomains) and save it to level-2-mail.ru.txt file
You can repeat the process by extracting whole levels

Then what to do next ?

You got two choices.

1- To get the live domains and complete your methodology

or

2- Complete your methodology while repeating the process with different level of domains starting from subdomains to the subdomains of subdomains etc. This gonna let you access domains no one visit it before and make your scope and chances bigger

# Conclusion
At the end you can use amass and other tools or even all tools you find. Cause as you saw each tool get new different domains which you was gonna miss. Have a great enumeration.
