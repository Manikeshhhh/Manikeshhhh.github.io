---
title: RCE,SSRF-to-EC2takeover and dropbox account takeover
published: true
---

### Description
So in this Blog post i will walk you through some of high severity issues i found in one of the Public Program. This was a company which was providing lots of SAAS applications. During this period I found several issues like IDOR on unauth API,HTMLI and many medium severity issue but I will mostly be talking about some of the interesting issue i found during this period.

### Inital Recon
I don't recon much initally and I will definitely improve my recon soon,even during recon phase i don't automate stuffs, I don't use nuclie either most of the time(sometimes running nuclie can really give you fruitful result but i don't always do), sometimes i also use gauplus to find some url. 
so, I always start of with subdomain enumeration.First i find some doamin from subfinder then i bruteforce to find some non listed sub-domain using ffuf.

```
root@x71n0:~$ subfinder -d xxx.com | httpx --status-code
```
I grep subdomains from here first then I use ffuf to bruteforce some new domains
```
root@x71n0:~$ ffuf -u https://FUZZ.xxxx.com -w bruteforcelist.txt -ac -mc 200,302,403
```
sometimes i use this fancy crawling tools like Katana to crawl from JS file(I usually look for vulnerable functions or secrets in JS file), I will talk about finding cloud assets when I talk about the **DropBox takeover** and I will talk about how i use Shodan when talking about the **SSRF**.

###  Finding my first ever RCE with LaTex injection

LaTeX is a document preparation system for high-quality typesetting. It is most often used for medium-to-large technical or scientific documents but it can be used for almost any form of publishing.LaTeX is based on the idea that it is better to leave document design to document designers, and to let authors get on with writing documents. So, in LaTeX you would input this document as:
```
\documentclass{article}
\title{Cartesian closed categories and the price of eggs}
\author{Jane Doe}
\date{September 1994}
\begin{document}
   \maketitle
   Hello world!
\end{document}

OUTPUT:

    This document is an article.
    Its title is Cartesian closed categories and the price of eggs.
    Its author is Jane Doe.
    It was written in September 1994.
    The document consists of a title followed by the text Hello world!

```
learn more about LaTex [Here](https://www.latex-project.org/about/)
So i found a subdomain **https://doctex.proc.xxx.com**, this application had many feature like managing team,importing CSV file,PPT,DOC,TEX,XLSX.when ever i see XlSX first i always try XXE, so as always i tried but this didn't work next i had no idea what is this tex file so i tried learning about latex, while reading a blog post on medium i found a article that talks about RCE with latex injection. I started reading about it and i found this [repo](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LaTeX%20Injection) which had payload of latex injection. 
The application was providing sample .tex file, i downloaded and this extra payload lines in the end.
```
\newread\file
\openin\file=/etc/passwd
\read\file to\line
\text{\line}
\closein\file
```
when i opened the converted pdf file,half of Documents was processed properly but more than half part of the document became white, next i saw payload which was encoding the response to base64
```
\input|ls|base64
```
this throwed a huge base64 data in the exported pdf and i decoded the base64 and it had response of my **ls** command. I tried some other command with base64
encoded and it worked. 

After reading about LaTex for sometime i found out that LaTeX does provide powerful programming features, so application should use industry-standard containerization and virtualization technologies to isolate your project from other projects on the same server.So, this is the end of RCE, next lets talk about the SSRF to EC2 takeover.


