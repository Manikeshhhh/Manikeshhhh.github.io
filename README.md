## whoami
I am Manikesh, I love to play CTFs,Pentest Web-applications,APIs and container security.
Currently I am learning Golang(Trying to build some Appsec tools) and tweaking Web-applications.
Other than Security and development, I love Playing PC Games,watch shows,anime,movies,Sometimes I also read Books.

This days i am also trying to touch Grass and make some friends :P

I will mainly be using this space to Publish Security research and writeup's
### 1. privilege escalation from User to Owner due to Mass assignment issue 
**Published on 23-Nov-2023**

This is a issue I found few months back on a program which was sadly duplicate.

### Description
User who has permission to invite other users to Org can invite any user as Owner and takeover the Organization.
I was also able to bypass the limit set to free that a Org can have only 2 Users.

### Thinking Process 
This was a heavily tested Target with more than 120+ resolved reports, which was quite a lot for a Program launched in 2021.
so without any expections i started testing the features,I greped some Js file and tried reading but sadly no luck(I look for secrets,weird functions and API keys). I Usually don't test for XSS's so I tried SSTI's on user inputs, I also tried to find some Open cloud storage and I found some s3 buckets but sadly no
luck because it only had read permission with some static files in it, so didn't report because it wasn't causing any security impact because those 
static files were anyways Public so it would definitely be so stupid of me to report.

Then I found a open google storage bucket with some employee's github access token from 2017, sadly any of those tokens were not valid so didn't report
this too. I almost gave up because i was fairly new to testing heavily tested Web applications. This looked like a hardend web-application.Then i left and came back to it the next day.

BTW, If you are wondering how was I able to Pull so many cloud assets,I am soon going to write a blog on that as well soon.
### The Issue 

Next day, I built the courage to test the features of main application again, I saw that this application was limiting free users to only add 
2 other users to orgnization. so I tried to Bypass it and I was successfull by passing user parameter as an array while inviting because of which i was able to bypass the Limit, I also tried some other Jutsu's too to bypass but this was only working way.

The request body looked like this
'''
{"scope":"xxxx","user":"xxxx@xxx.com","roles":["xxx.User"]}
I tried passing user parameter as an array and it worked
{"scope":"xxxx","user":["xxxx@xxx.com","xxx2@xxx.com","xxx3@xx.com"],"roles":["xxx.User"]}
'''
I thought not to report this issue because i was pretty sure this would be duplicate and It was already 3am so i thought to Get some sleep and try to find some new assets tomorrow.

the next day, I logged in to the lower privilege user can tried if i could see the Owner data and thankfully I was able to see, I saw some new fields so I thought to pass those fields during adding and new User and voilla!!,just like that I took over the organizaion as the lowest privilege users.

so i added some extra data with the request body, you can see below
'''
{"scope":"xxx","user":"xxx@xx.com","roles":["xxxx.Creator","xxxx.Owner"]}
'''

Thank you for taking your time to read my Blog,
This was it. If you liked the blog,you can follow me on my handles.

## Contact 
[Instagram](https://www.instagram.com/manikeshh/)  [Twitter](https://twitter.com/X71n0/)  [Github](https://github.com/Manikeshhhh)
[Email](offsecmanikesh@gmail.com)
