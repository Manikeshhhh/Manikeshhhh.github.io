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

Not yet completed.....................

