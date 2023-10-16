---
title: A Bucket Takeover Vulnerability with High-Impact Consequences
published: true
---

### Description
I will keep it simple because the Vulnerability is pretty straight forward, It was the exploitation and impact which was interesting. The impact of the issue was critical because it was impacting the admins of the organization. This was on one of the sony assets, I don't have full discloser permission so i will be masking some of the details in this blog.

I could have uploaded link to my phising page or could have hosted malicious links for the organization members to download, which would have serious impact.

For POC purpose, This is what the user would see if he would have visited any page that does not exist in the application.

![image](https://github.com/Manikeshhhh/Manikeshhhh.github.io/assets/88855149/4cd84816-05c7-4f39-81d2-67759b739659)



### Thinking Process 

Initially since sony has huge target. I started with subdomain enumeration and came across one of the interesting application that was used **internally** and i found that the application did not have any signup page so i tried sending some login request and from my proxy logs, i found out that the application was using **amazon cognito** under the hood for authentication so I tried to download the **main.js** file and tried to find all the required pieces like the **region,poolID,client,etc** and I did a thorough test for Cognito vulnerbility like trying to create a account manually but there was proper check in place in cognito too.

After all this i was trying to go through my logs once again before going to bed and i saw something interesting.

### Exploitation

so, while i was going through the request, I found another **JS file** in the logs, so i started going through the JS code and I found that on error the application was serving a page from a s3 bucket. So, I tried to visit the url to check if a open s3 bucket or do i have any other permission on the application but on visiting the **s3 bucket** url refrenced in the Js file i was served a bucket not exist page.

![image](https://github.com/Manikeshhhh/Manikeshhhh.github.io/assets/88855149/30c51a72-11d7-4471-a545-02101329b494)

so immediately i found a created a **s3 bucket with same name and region** and removed all the authentication from my s3 bucket and made it open so the application will be able to load it without any problem. I added the same folders and then the file with same name and created the bucket.

Now when i tried going to the any non existent page on the application, I was able to see this page.

![image](https://github.com/Manikeshhhh/Manikeshhhh.github.io/assets/88855149/b801ed8a-a829-46ab-acfc-d26d1038c74a)




### Impact 

 **Malware Distribution**: The attacker could host and distribute malware, such as viruses, ransomware, or spyware, via the compromised bucket. Any user who unknowingly downloads these files could put their system and the organization's data at risk.

  **Phishing Attacks**: With control over the bucket, the attacker could upload phishing pages, creating convincing replicas of legitimate web pages, and lure unsuspecting users into revealing their credentials or sensitive information.


 **Operational Disruption**: The unauthorized control over assets could disrupt critical business operations, causing downtime and financial losses because they might be refrencing the same bucket in some other applications as well which they might have deleted later on unknowningly 

**Reputation Damage**: The compromised bucket could be used to deface the organization's web assets, tarnishing its reputation and trustworthiness among customers and partners.



## Timeline

reported -  17-07-23
Triaged  -  21-07-23
Resolved -  20-08-23
Awarded with swag -  25-08-23

### takeaway 
This incident serves as a stark reminder of the importance of thorough security assessments and continuous monitoring. 


Thank you for taking your time to read my Blog,
This was it. If you liked the blog,you can follow me on my handles.



## Contact 
[Instagram](https://www.instagram.com/manikeshh/)  [Twitter](https://twitter.com/X71n0/)  [Github](https://github.com/Manikeshhhh)
[Email](offsecmanikesh@gmail.com)
