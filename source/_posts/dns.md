---
title: All You Need to Know About DNS
date: 2021-03-06
tags: computer networking, DNS, domain
index_img: /images/thumbnail/dns.png
---
### Things behind a domain 
We all know that to visit a website, you have to provide browser with the domain name of that website. However, it is not the domain name that the browser sends the http request with to retrieve the website content, instead, it is the ip address of the hosting server that hosts the website you want to visit. A domain is basically the translation of an ip address of a web hosting server. An analogy that is popularly used is phonebook. To make a call to someone on your cell-phone, normally you don't memorize the actual phone number of that person. Instead, you go to the contact list on your cell-phone, find the contact name, and make the call. An ip address is like the phone number, and the domain would be the contact name you alised for that ip address. 
```bash
$ ping doerj.github.io

Pinging doerj.github.io [185.199.109.153] with 32 bytes of data:
Reply from 185.199.109.153: bytes=32 time=26ms TTL=60
Reply from 185.199.109.153: bytes=32 time=15ms TTL=60
Reply from 185.199.109.153: bytes=32 time=12ms TTL=60
Reply from 185.199.109.153: bytes=32 time=12ms TTL=60
```  
The above command sends ping to the server that is hosting my personal website `doerj.github.io` which is a domain name. And the following replies are shown as coming from `185.199.109.153`, which is the ip address of the hosting server for my website. 

But what actually happens when you type a domain in the url bar of the browser? This is where DNS plays a significant role. DNS(Domain Name System) is an infrastructure that matches the domain provided by the browser with the corresponding ip address. To put it simple, DNS helps you find the ip address for the website you want to visit. 

##### DNS caching
When you type in a domain in the url bar of the browser, the first thing your browser does is to search the ip address of that domain in your computer's cache memory, known as DNS caching. The following terminal snippet shows a DNS cache record for the domain of my personal website `doerj.github.io`, stored on my local machine. As you can see, there is a parameter TTL(Time To Live) indicating the time duration this DNS cache record will remain in the cache memory. 
```bash
    doerj.github.io
    ----------------------------------------
    Record Name . . . . . : doerj.github.io
    Record Type . . . . . : 1
    Time To Live  . . . . : 389
    Data Length . . . . . : 4
    Section . . . . . . . : Answer
    A (Host) Record . . . : 185.199.109.153
```
Once TTL is passed, the DNS lookup will be redirected to the ISP(Internet Service Provider) server for your network, which is also known as the DNS resolver for you local machine. DNS resolver will first check its own cache memory to see if the ip address for the requested domain is still remained, if not, the DNS resolver will send the lookup request to the DNS nameservers. 

##### DNS nameservers
DNS nameservers are a set of servers which stores the information about different portions of a domain name. The nameservers on the top of the hierarchy are called root nameservers. There are 13 root nameservers in total distributed worldwide, taking the DNS lookup requests from DNS resolvers, parse the root domain, for example `doerj.github.io`, and redirect the request to lower-level nameservers which is TLD nameservers.

TLD stands for Top-Level Domain, which refers to the portion after the last period "." in a domain, such as ".com". It indicates what organization the domain is issued by and under what authority the domain should be operated. TLD nameservers store information about all the websites that end with different TLD, for example, .COM TLD nameserver stores information about all the domain names that have extension ".com". When the TLD nameserver receives the request from the root nameserver(use `github.doerj.io` as an example), it queries the ip address of the nameserver that stores the authoritative part of the domain(`doerj.io`), and sends query to that authoritative nameserver.

An authoritative nameserver, also known as a second-level nameserver, stores the actual ip address of the web server hosting the web content you are requesting for. To understand the role of an authoritative nameserver in DNS, we have to be aware of what happened when a customer purchase a domain. When you buy a domain name from a registrar, you are actually buying the right to control the response sent from the TLD nameserver. Specifically, you get the right to decide which authoritative nameserver to use for finding the final ip address of the domain. For example, Google has the ownership of domain `www.google.com`, therefore, Google has the right to point the response from .COM TLD nameserver to Google's own nameservers.

Once the ip address of that domain is found by the authoritative nameserver, the ip address will be returned to the DNS resolver, and finally reaches to the browser to initialize the http request for fetching the web content resources. 

### A closer look to the authoritative nameserver
To have insight of what's in a authoritative nameserver, we need to understand what a DNS zone is. DNS zones basically refers to any distinct part of a domain which is delegated to an entity such as an organization, a company, or even a person. For example, `.io`, which is one of the DNS zoens for `github.doerj.io`, is delegated to British Indian Ocean Territory. And as another DNS zone, `doerj.io` is assigned to the organization or company that operates its authoritative nameserver. 

For each DNS zone stored on an authoritative nameserver, it has a corresponding DNS file that contains all the DNS records for that DNS zone. A DNS record tells the infomation about a domain including the hosting ip address of that domain, and how to handle the query request for the domain. The following screenshot is capture from a registrar's control panel, and it shows all the DNS records associated with a DNS zone. 
![Site Image](/images/dns/dns-record.png)
As you can see, there are different types of DNS record. The ones that are mostly used are `A record` and `CName record`. A record specifies the hosting ip address for the domain, while CName recore defines the forwarding behavior for any subdomain that is associated with the domain. For example, `wiki.example.com` is a subdomain for `example.com`, and the CName record in this case will specify which subdomains will be forwarded to `example.com`.

Note that there is also TTL for each DNS record, indicating how long the record will be cached after each query for that DNS zone. Therefore, when you switch the hosting server for a website, the server update will not be reflected immediately, because the DNS record for the domain of the website is still in the cache and pointing to the old hosting server. Furthermore, if your DNS record has been updated and points to the new hosting server, it doesn't necessarily mean the other visitors will get the update by then since every visitor may use a different authoritative nameserver for querying the domain of your website. 

### Security measure 
Initially, the DNS lookup process was not validated with any security measure, that is, there is no way to confirm that the returned ip address exactly matches with the queried domain. Imagine the authoritative nameserver for some domains are hacked by malicious third-party, and all cached DNS records are manipulated. Then the ip address returned to the DNS resolve may leads user to the sites made by the hackers. 

In order to ensure the returned ip address is trusted for the domain user is requesting for, `DNSSEC` is introduced. DNSSEC stands for DNS Security Extensions, and it aims to build a chain of trust among the hierarchy of DNS nameservers by adding digital signature to the DNS data(i.e., the response from DNS nameservers) and the organization who is in charge of each level's DNS nameserver verifies whether the signature is valid. If valid, then the DNS data can be trusted and returned to the DNS resolver. Otherwise, the ip address could be potentially malicious. 

When user buy a new domain from any registrar, for example, Godaddy.com, a DNS zone for that domain will be created and a pair of public/private keys will be generated. The private key is used to sign the digital signature for the DNS data, and the signature will be stored in `RRSIG(Resource Record Signature)` and sent in the DNS query response. Along with RRSIG, the `DNSKEY` which contains the public key will also be included in the DNS data, and other nameservers(i.e., TLD nameserver) will use this public key to verify the signature. The chain goes on for TLD and root nameservers, so on and so forth, until the DNS data(i.e., the ip address for the requested domain) is returned to the DNS resolver.

These are all the basic things you need to know about DNS. Hope you found this article useful, and I'll see you next time!


