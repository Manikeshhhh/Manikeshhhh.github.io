---
title: 'The Poisonous Cache: Unmasking a Stealthy Threat'
published: true
---

### Description
In this comprehensive article, we will explore the fascinating topic of Web cache poisoning, delving into the nature of Web caches and their potential vulnerabilities for exploitation. With real-world examples as our guide, we will uncover the intricacies of this subject and shed light on various techniques to exploit Web caches effectively.Throughout the article, we will examine notable case studies and provide insights into bypassing certain technologies like Cloudflare, AKAMAI, and Cloudfront.For this article I will take a alot of reference from James kettle research papers.

### Please come back later as this article still hasn't been completed 

### Fundamentals and Terminologies:

**Cache is a form of auxiliary memory that holds data and instructions that an app or website needs regular access to. It’s fast, easy to access, and allows whatever application or website that uses the files in the cache to work faster.**

There are several types of caches for e.g. : 

1. **Memory Cache**

  - Primary Cache L1
  - Secondary Cache L2
  - Main Memory L3 Cache

Some of these caches work directly with your device’s central processing unit (CPU). Others are external caches that store data away from the CPU but allow for easy access when needed. But all focus on caching data that your device requires to run quickly and correctly.

2. **Web Cache**

This is also type of cache which we are going to talk about in this paper

Have you ever wondered why a web page that you visit often tends to load faster than a page you’ve never visited before?
You can thank web caches for that.

Web caches store data from browsers, websites, and servers that allow them to quickly access information needed to speed up loading times. Without web caching, your browser has to send a new request every single time you access a web page.
There are four main types of web cache:

  1. Site Cache
  2. Browser Cache
  3. Micro Cache
  4. Server Cache

I don’t want to make this paper caching 101 so you can read more about caching [here](https://softwarelab.org/blog/what-is-a-cache/), coming to other types of caches there are generally 6 types of caches.

3. Application/Software Caches
4. Data Caching
5. Application/Output Caches
6. Distributed Caching


### How web Caching works:

 Websites often tend to use web cache functionality (for example over a CDN, a load balancer, or simply a reverse proxy). The purpose is simple: store files that are often retrieved, to reduce latency from the web server.
 
Let's see an example of web cache. Website [http://www.example.com](http://www.example.com/) is configured to go through a reverse proxy. A dynamic page that is stored on the server and returns personal content of users, such as http://www.example.com/home.php, will have to create it dynamically per user, since the data is different for each user. This kind of data, or at least its personalized parts, isn't cached.

What's more reasonable and common to cache are static, public files: style sheets (css), scripts (js), text files (txt), images (png, bmp, gif), etc. This makes sense because these files usually don't contain any sensitive information. In addition, as can be found in various best practices articles about web cache configuration, it's recommended to cache all static files that are meant to be public, and disregard their HTTP caching headers.

Web caching is a core design feature of the HTTP protocol meant to minimize network traffic while improving the perceived responsiveness of the system as a whole. Caches are found at every level of a content’s journey from the original server to the browser.

Web caching works by caching the HTTP responses for requests according to certain rules. Subsequent requests for cached content can then be fulfilled from a cache closer to the user instead of sending the request all the way back to the web server.

In simple terms, a caching server acts as a middleman between your browser and the application server. Its job is to store a copy of the files that the application server provides to users and then deliver those files directly to the user when requested, without involving the application server every time.

This diagrams shows how web caches works:
![image](https://github.com/Manikeshhhh/Manikeshhhh.github.io/assets/88855149/6fec4a4e-42ab-49b8-8b54-4d2c9980be7a)
credits: Port-swigger
 The caching server works like a temporary storage unit, holding commonly accessed files to make future requests faster and reduces the load on application server.
 
 **What can be cached?**

Certain content lends itself more readily to caching than others. Some very cache-friendly content for most sites are:

- Logos and brand images
- Non-rotating images in general (navigation icons, for example)
- Style sheets
- General JavaScript files
- Downloadable Content
- Media Files

**How to setup a caching server for your application?**

Some companies choose to host their own cache using software such as Varnish, while others opt to rely on a Content Delivery Network (CDN) like Cloudflare, which has distributed caches across geographical locations. Additionally, some popular web applications and frameworks like Drupal have built-in caching functionality. 

**Note: From here I will be using some part of explanation from James kettle research paper from 2018 because he has done excellent Job explaining Few key points.**

### What are cache keys:

![image](https://github.com/Manikeshhhh/Manikeshhhh.github.io/assets/88855149/e612c5a5-8a25-44c3-9fa8-72465b7bdd4f)

The concept of caching might sound clean and simple, but it hides some risky assumptions. Whenever a cache receives a request for a resource, it needs to decide whether it has a copy of this exact resource already saved and can reply with that, or if it needs to forward the request to the application server.

Identifying whether two requests are trying to load the same resource can be tricky; requiring that the requests match byte-for-byte is utterly ineffective, as HTTP requests are full of inconsequential data, such as the requester's browser:

```
*GET /blog/post.php?mobile=1 HTTP/1.1
Host: example.com*
User-Agent: Mozilla/5.0 … Firefox/57.0
Accept: */*; q=0.01
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://google.com/
Cookie: jessionid=xyz;
Connection: close
```
Caches tackle this problem using the concept of cache keys – a few specific components of a HTTP request that are taken to fully identify the resource being requested. In the request above, I've highlighted the values included in a typical cache key in marked with star.

This means that caches think the following two requests are equivalent, and will happily respond to the second request with a response cached from the first:

```
*GET /blog/post.php?mobile=1 HTTP/1.1
Host: example.com*
User-Agent: Mozilla/5.0 … Firefox/57.0
Cookie: language=pl;
Connection: close
```

```
*GET /blog/post.php?mobile=1 HTTP/1.1
Host: example.com*
User-Agent: Mozilla/5.0 … Firefox/57.0
Cookie: language=en;
Connection: close
```
As a result, the page will be served in the wrong language to the second visitor. This hints at the problem – any difference in the response triggered by an unkeyed input may be stored and served to other users. In theory, sites can use the 'Vary' response header to specify additional request headers that should be keyed. in practice, the Vary header is only used in a rudimentary way, CDNs like Cloudflare ignore it outright, and people don't even realize their application supports any header-based input.

This causes a healthy number of accidental breakages, but the fun really starts when someone intentionally sets out to exploit it.


### Now let’s talk about the research Topic:  Cache poisoning

Cache Poisoning refers to when a user sends a request which causes harmful/malicious response being cached into the server and being served to other users of the application 

![image](https://github.com/Manikeshhhh/Manikeshhhh.github.io/assets/88855149/38a527c6-d199-480f-a3b6-101319e6770d)
credits: Hackmanit

In the above diagram case, the referral parameter is vulnerable to RXSS and referral header is also not the part of unkeyed, hence it won’t be considered while serving to other potential users of the application. If proper defense does not exist this can also cause mass account takeover in the application.

### Exploiting web cache poisoning:

| Title | Tags | Programs | Authors | Bounty | Publication Date | Added Date |
| --- | --- | --- | --- | --- | --- | --- |

|  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- |
| https://medium.com/@zhero_/dos-via-cache-poisoning-38f3a87f997c | Web cache deception, DoS | - | Allam Rachid (@blank_cold) | - | 2023-05-17 | 2023-05-18 |
| https://medium.com/tinder/identifying-vulnerabilities-in-github-actions-aws-oidc-configurations-8067c400d5b8 | CI/CD, OpenID Connect | AWS | Rojan Rijal (@uraniumhacker), Johnny Nipper (@ratherbeonline), Tanner Emek (@itscachemoney) | - | 2023-04-18 | 2023-04-24 |
| https://medium.com/@cachemoney/exploiting-application-logic-to-phish-internal-mailing-lists-486b94fc2ef1 | Phishing | - | Tanner Emek (@itscachemoney) | - | 2023-01-13 | 2023-03-10 |
| https://github.com/AnkitCuriosity/Write-Ups/blob/main/Web%20Cache%20Poisoning%20-%20Capability%20to%20disable%E2%88%95deface%20the%20app.vulnerable.com%20(A%20tale%20of%20poisoning%20through%20the%20layers%20of%20caching).md | Web cache poisoning | - | Ankit Singh (@AnkitCuriosity) | $1,000 | 2023-03-03 | 2023-03-06 |
| https://medium.com/@snoopy101/web-cache-deception-attack-on-a-private-bug-bounty-program-52872cbdeedc | Web cache deception | - | snoopy (@snoopy101101) | - | 2023-03-01 | 2023-03-06 |
| https://blog.quarkslab.com/post-exploitation-abusing-the-keepass-plugin-cache.html | Local Privilege escalation, Windows | KeePass | Kevin Minacori | - | 2023-02-07 | 2023-02-13 |
| https://enumerated.wordpress.com/2020/08/05/the-case-of-the-missing-cache-keys/ | Web cache poisoning | - | Aaron Costello (@ConspiracyProof) | - | 2020-08-05 | 2023-02-13 |
| https://mazoka777.medium.com/how-i-hacked-all-zendesk-sites-265-000-site-by-one-line-c6b6485a7a6 | Web cache poisoning | Zendesk | Ahmed Salah Abdalhfaz (@Elsfa7-110) | - | 2023-01-30 | 2023-01-31 |
| https://www.zerodayinitiative.com/blog/2023/1/23/activation-context-cache-poisoning-exploiting-csrss-for-privilege-escalation | Local Privilege Escalation, Windows | Microsoft | Simon Zuckerbraun | - | 2023-01-23 | 2023-01-26 |
| https://yaseenzubair.medium.com/web-cache-poisoning-worth-it-e7c6d88797b1 | Web cache poisoning, XSS | - | Yaseen Zubair | $200 | 2023-01-02 | 2023-01-06 |
| https://spyclub.tech/2022/12/14/unusual-cache-poisoning-akamai-s3/ | Web cache poisoning, Host header injection | Akamai | SpyD3r (@TarunkantG) | - | 2022-12-14 | 2022-12-15 |
| https://github.com/0xacb/recollapse/blob/main/till_recollapse_fuzzing_the_web_for_mysterious_bugs.pdf & https://0xacb.com/2022/11/21/recollapse/ | Regex, Account takeover, Open redirect, Web cache deception, Buffer Overflow, OAuth, Normalization | - | André Baptista (@0xacb) | - | 2022-11-21 | 2022-11-23 |
| https://www.akamai.com/blog/security-research/cold-hard-cache-bypassing-rpc-with-cache-abuse | Privilege escalation, Windows | Microsoft | - | - | 2022-10-11 | 2022-10-17 |
| https://sec-consult.com/blog/detail/melting-the-dns-iceberg-taking-over-your-infrastructure-kaminsky-style/ | DNS cache poisoning, Kaminsky attack | - | Timo Longin, Clemens Stockenreitne | - | 2022-10-06 | 2022-10-06 |
| https://medium.com/@jacopotediosi/worldwide-server-side-cache-poisoning-on-all-akamai-edge-nodes-50k-bounty-earned-f97d80f3922b | Web cache poisoning | Akamai, Paypal, Airbnb, Tesla, Valve, Zomato, Whitejar, Starbucks, PlayStation, Marriott, Hyatt Hotels, Goldman Sachs, Microsoft, Apple, LastPass, Brussels Airlines, Mastercard, eToro BBP, BMW Group, Rockstar Games | Francesco Mariani (@_medusa_1_), Jacopo Tediosi (@jacopotediosi) | $50,000 | 2022-09-29 | 2022-10-02 |
| https://samcurry.net/universal-xss-on-netlifys-next-js-library/ | Universal XSS, SSRF, Open redirect, Web cache poisoning | Netlify, Gemini, PancakeSwap, Docusign, Moonpay, Celo | Sam Curry (@samwcyo) | - | 2022-09-21 | 2022-09-22 |
| https://nokline.github.io/bugbounty/2022/09/02/Glassdoor-Cache-Poisoning.html & https://hackerone.com/reports/1621540 | Web cache poisoning, XSS, DoS | Glassdoor | Harel (@h4r3l) | $1,700 | 2022-09-02 | 2022-09-15 |
| https://med-mahmoudi26.medium.com/saving-more-than-100-000-website-from-a-watering-hole-attack-a22f63a37f94 | Web cache poisoning, Watering hole attack | HubSpot | mohamad mahmoudi (@Lotus_619) | $5,000 | 2022-08-31 | 2022-09-15 |
| https://blog.orange.tw/2022/08/lets-dance-in-the-cache-destabilizing-hash-table-on-microsoft-iis.html | DoS, Web cache poisoning, Authentication bypass | Microsoft | Orange Tsai (@orange_8361) | $30,000 | 2022-08-18 | 2022-09-15 |
| https://www.usenix.org/conference/usenixsecurity22/presentation/mirheidari | Web cache deception | - | Seyed Ali Mirheidari, Matteo Golinelli, Kaan Onarlioglu, Engin Kirda, Bruno Crispo | - | 2022-08-10 | 2022-09-15 |
| https://i.blackhat.com/USA-22/Wednesday/US-22-Doyhenard-Internal-Server-Error-wp.pdf & https://i.blackhat.com/USA-22/Wednesday/US-22-Doyhenard-Internal-Server-Error.pdf | Memory corruption, RCE, HTTP Request Smuggling, Web cache poisoning, Desync attack | SAP | Martin Doyhenard (@tincho_508) | - | 2022-08-10 | 2022-09-15 |
| https://medium.com/tinder/exploiting-github-actions-on-open-source-projects-5d93936d189f | RCE | Elastic | Rojan Rijal (@uraniumhacker), Johnny Nipper (@ratherbeonline), Tanner Emek (@itscachemoney) | - | 2022-07-26 | 2022-09-15 |
| https://bxmbn.medium.com/how-i-test-for-web-cache-vulnerabilities-tips-and-tricks-9b138da08ff9 | Web cache poisoning, Web cache deception | - | Kevin (@bxmbn) | $3,500 | 2022-07-21 | 2022-09-15 |
| https://www.sonarsource.com/blog/zimbra-mail-stealing-clear-text-credentials-via-memcache-injection/ | Memcache injection, CRLF injection | Zimbra | Sonar (@SonarSource) | - | 2022-06-14 | 2022-09-15 |
| https://medium.com/@_ip_/3-3-cache-poisoning-lateral-movement-gitlab-9c6288708576 | Broken Access Control | GitLab | IP | - | 2022-04-15 | 2022-09-15 |
| https://www.rapid7.com/blog/post/2022/04/12/cve-2022-24527-microsoft-connected-cache-local-privilege-escalation-fixed/ | Local Privilege Escalation | Microsoft | Jacob Baines (@Junior_Baines) | - | 2022-04-12 | 2022-09-15 |
| https://scribesecurity.com/blog/github-cache-poisoning | Cache poisoning attack, Logic flaw | GitHub | Scribe Security (@ScribeSecurity) | - | 2022-03-30 | 2022-09-15 |
| https://bxmbn.medium.com/how-i-made-15-000-by-hacking-caching-servers-part-1-5541712a61c3, https://bxmbn.medium.com/how-i-made-16-500-hacking-cdn-caching-servers-part-2-4995ece4c6e6 & https://infosecwriteups.com/how-i-made-16-500-hacking-cdn-caching-servers-part-3-91f9d836e046 | Web cache poisoning, Stored XSS, Web cache deception | - | Kevin (@bxmbn) | $16,500 | 2022-01-29 | 2022-09-15 |
| https://www.tldr.engineering/how-i-found-and-fixed-a-vulnerability-in-python/ | Web cache poisoning | Python | Adam Goldschmidt (@AdamGolds) | - | 2021-12-24 | 2022-09-15 |
| https://youst.in/posts/cache-poisoning-at-scale/ | Web cache poisoning | GitHub, GitLab, HackerOne, Shopify, Cloudflare | Youstin (@iustinBB) | $40,000 | 2021-12-23 | 2022-09-15 |
| https://thalium.github.io/blog/posts/leaking-aslr-through-rdp-printer-cache-registry/ | Memory corruption | Microsoft | Valentino Ricotta | $1,000 | 2021-12-10 | 2022-09-15 |
| https://medium.com/@priyanshbansal25/unauthenticated-cache-purge-c56fac8569e8 | Unauthenticated cache purge | Lenovo | Priyansh Bansal (@PriyanshB25) | - | 2021-10-28 | 2022-09-15 |
| https://www.zerodayinitiative.com/blog/2021/9/2/cve-2021-2429-a-heap-based-buffer-overflow-bug-in-the-mysql-innodb-memcached-plugin | Memory corruption | Oracle (MySQL) | - | - | 2021-09-02 | 2022-09-15 |
| https://web.archive.org/web/20210829191303/https://0u.ma/5 | XSS, Web cache poisoning | - | ElMahdi Mrhassel (@ElMrhassel) | - | 2021-08-28 | 2022-09-15 |
| https://sapt.medium.com/apple-hall-of-fame-for-a-small-misconfiguration-unauth-cache-purging-faf81b19419b | Unauthenticated cache purge | Apple | Prajit Sindhkar (@PrajitSindhkar) | - | 2021-07-26 | 2022-09-15 |
| https://blog.lbherrera.me/posts/appcache-forgotten-tales/ | Browser hacking | Google (Chrome) | Luan Herrera (@lbherrera_) | $10,000 | 2021-05-31 | 2022-09-15 |
| https://n3t-hunt3r.medium.com/finding-my-first-critical-web-cache-poisoning-6f956799371c | Web cache poisoning | - | Yasser Khan (@N3T_hunt3r) | - | 2021-05-18 | 2022-09-15 |
| https://robertchen.cc/blog/2021/04/03/github-pages-xss | XSS, CRLF injection, Web cache poisoning | GitHub | Robert Chen (@NotDeGhost), Philip | $35,000 | 2021-04-04 | 2022-09-15 |
| https://blog.melbadry9.xyz/fuzzing/nuclei-cache-poisoning | Web cache poisoning, Stored XSS | - | Mohamed Elbadry (@_melbadry9) | $1,500 | 2021-04-02 | 2022-09-15 |
| https://galnagli.com/Cache_Poisoning/ | Web cache poisoning, Stored XSS | - | Gal Nagli (@naglinagli) | $1,000 | 2021-02-25 | 2022-09-15 |
| https://jsecu.github.io/2021/02/21/poisoning/ | Web cache poisoning, Account takeover | - | Josh Fam (@Pullerze) | - | 2021-02-21 | 2022-09-15 |
| https://ysamm.com/?p=629 | Information disclosure, Caching issue | Meta / Facebook | Youssef Sammouda (@samm0uda) | $4,800 | 2021-02-17 | 2022-09-15 |
| https://pullerjsecu.medium.com/how-i-was-able-to-turn-a-xss-into-a-account-takeover-ae0c478640e7 | Web cache poisoning, Stored XSS, Account takeover, OAuth, Logic flaw | - | Josh Fam (@Pullerze) | - | 2021-02-03 | 2022-09-15 |
| https://web.archive.org/web/20210730144815/https://www.cysek.org/post/sxss-by-cache-poison-attack | Web cache poisoning, Stored XSS | - | Schizo! | - | 2021-01-14 | 2022-09-15 |
| https://iustin24.github.io/Cache-Key-Normalization-Denial-of-Service/ | Web cache poisoning, DoS | - | Youstin (@iustinBB) | - | 2020-12-29 | 2022-09-15 |
| https://lutfumertceylan.com.tr/posts/acc-takeover-web-cache-xss/ | Reflected XSS, Web cache poisoning, Account takeover | - | Lütfü Mert Ceylan (@lutfumertceylan) | - | 2020-12-26 | 2022-09-15 |
| https://web.archive.org/web/20201125190336/https://tox7cv3nom.github.io/2020-08-05-how_i_was_able_to_pawned_website_via_escilating_webcache-deception-to-rce/ | Web cache deception, SSRF, RCE | - | mohit (@mohit29295572) | - | 2020-09-05 | 2022-09-15 |
| https://medium.com/bugbountywriteup/cache-poisoning-of-wget-94a4d70104b1 | Web cache poisoning | - | Vuk Ivanovic | - | 2020-08-12 | 2022-09-15 |
| https://medium.com/bugbountywriteup/cache-poisoning-with-xss-a-peculiar-case-eb5973850814 | XSS, Web cache poisoning | - | Vuk Ivanovic | - | 2020-08-08 | 2022-09-15 |
| https://web.archive.org/web/20200426140225/https://medium.com/@aungpyaehackeronetester/web-cache-poisoning-in-postmates-1500-a67eee4fc118 | Web cache poisoning | Postmates | Aung Pyae Ko Ko (@BlcKVRtuL1) | $1,500 | 2020-04-24 | 2022-09-15 |
| https://samcurry.net/abusing-http-path-normalization-and-cache-poisoning-to-steal-rocket-league-accounts/ | HTTP cache poisoning, Open redirect | Rocket League | Sam Curry (@samwcyo) | - | 2020-04-19 | 2022-09-15 |
| https://medium.com/@ozguralp/weird-vulnerabilities-happening-on-load-balancers-shallow-copies-and-caches-9194d4f72322 | Information disclosure | - | Ozgur Alp (@ozgur_bbh) | $1,500 | 2020-02-11 | 2022-09-15 |
| https://enumerated.wordpress.com/2019/12/24/sop-bypass-via-browser-cache | SOP bypass | Keybase | Aaron Costello (@ConspiracyProof) | $1,500 | 2019-12-24 | 2022-09-15 |
| https://terjanq.github.io/Bug-Bounty/Google/cache-attack-06jd2d2mz2r0/index.html | XS-Search | Google | Terjanq (@terjanq) | - | 2019-11-12 | 2022-09-15 |
| https://portswigger.net/research/responsible-denial-of-service-with-web-cache-poisoning | DoS, Web cache poisoning | Tesla, HackerOne, Deliveroo, Bitbucket, Paypal, Meta / Facebook, Twitter | James Kettle (@albinowax) | $22,300 | 2019-10-24 | 2022-09-15 |
| https://cpdos.org/ | DoS, Web cache poisoning | Microsoft, Amazon, Akamai, Cloudflare, Yahoo! / Verizon Media, Play Framework | Hoai Viet Nguyen (@hvnguyen86), Luigi Lo Iacono, and Hannes Federrath | - | 2019-10-22 | 2022-09-15 |
| https://medium.com/@nahoragg/chaining-cache-poisoning-to-stored-xss-b910076bda4f | Web cache poisoning, Stored XSS | - | Rohan aggarwal (@nahoragg) | - | 2019-07-28 | 2022-09-15 |
| https://medium.com/@dr.spitfire/sensitive-information-disclosure-web-cache-deception-attack-bcac6cb9cd86?sk=a2557f0c557ff38876141c2d94b296dd | Information disclosure | Intuit | Wasim Shaikh (@Wa_sim_sim) | - | 2019-06-26 | 2022-09-15 |
| https://medium.com/@logicbomb_1/the-journey-of-web-cache-firewall-bypass-to-ssrf-to-aws-credentials-compromise-b250fb40af82 | LFI, SSRF, WAF bypass, Cloudflare bypass | - | Avinash Jain (@logicbomb_1) | - | 2019-04-25 | 2022-09-15 |
| https://medium.com/@kunal94/web-cache-deception-to-api-endpoint-attack-using-cached-token-header-b01a604a5ccd | Web cache deception | - | Kunal pandey (@kunalp94) | $250 | 2019-04-13 | 2022-09-15 |
| https://medium.com/@kunal94/web-cache-deception-attack-leads-to-user-info-disclosure-805318f7bb29 | Web cache deception, Information disclosure | - | Kunal pandey (@kunalp94) | $300 | 2019-02-25 | 2022-09-15 |
| https://medium.freecodecamp.org/cache-deception-how-i-discovered-a-vulnerability-in-medium-and-helped-them-fix-it-31cec2a3938b | Web cache deception | Medium | Yuval Shprinz | $100 | 2019-02-06 | 2022-09-15 |
| https://portswigger.net/research/bypassing-web-cache-poisoning-countermeasures | Web cache poisoning | Cloudflare | James Kettle (@albinowax) | - | 2018-10-05 | 2022-09-15 |
| https://portswigger.net/research/practical-web-cache-poisoning | Web cache poisoning | Mozilla, HubSpot, Cloudflare, Binary.com, Amazon (CloudFront) | James Kettle (@albinowax) | - | 2018-08-09 | 2022-09-15 |
| https://medium.com/@cachemoney/using-a-github-app-to-escalate-to-an-organization-owner-for-a-10-000-bounty-4ec307168631 | Authorization flaw, IDOR | GitHub | Tanner Emek (@itscachemoney) | $10,000 | 2018-06-20 | 2022-09-15 |

