---
layout: post
title: The SPF Subdomain Issue
tags: DNS Security SPF Domains
---

While setting up some new DNS records for a mail gateway migration, I found myself reading the related RFCs which we all know tends to hurt the brain but this time it lead to something quite interesting. After reading the ins and outs of SPF, DKIM and DMARC I could see several common implementation scenarios that would allow carefully crafted emails to circumvent the implemented controls, as SPF doesn't apply to subdomains.

It might just be that 1 DNS A record needs another 2 TXT records to close the gaps. 

<!--more-->



## SPF 
SPF is a pretty straight forward concept - we simply publish a DNS TXT record that lists the permitted mail servers for a domain. 

```bash
hello.com.  3600  IN  TXT "v=spf1 ip4:4.3.2.1 include:spf.protection.outlook.com -all"
```

It's simple to implement and is extremely effective at what it does, however it is worth noting a couple things:
- The action taken (i.e. what to do when SPF fails) is completely dependant on the configuration of the receiving email gateway. 
- A missing / non-existent SPF record is often treated as a neutral result, which can be treated differently by different vendors, but is commonly treated as benign. 
- SPF lookups are for the exact domain and subdomains are **NOT** covered.

The last point here is the one to take the most note of...

As subdomains are not covered by a SPF record, emails sent from a subdomain (regardless of whether the subdomain exists or not) will be checked against an SPF record that doesn't exist. So how do we publish a record for a non-existent subdomain? That's where a wildcard record can help!   

## Wildcard Records

[RFC7208](https://datatracker.ietf.org/doc/html/rfc7208#section-3.5) has a really interesting section that alludes to the issue but doesn't clearly articulate the problem or requirement.

With the use of wildcard records we can return results when a request is made for a non-existent record. This allows us to start closing the gaps on the SPF subdomain issues by implementing a wildcard TXT record with either your standard SPF data or a SPF NULL (v=spf1 -all) to be returned for non-existent records. Here is an example of what that would look like:

```bash
# Protection for the base domain
example.com.  3600  IN  TXT	"v=spf1 ip:1.2.3.4 -all"

# Protect any non-existent subdomain
*.example.com.  3600  IN  TXT	"v=spf1 -all"
```

It's important here to think about wildcard records correctly. The "*" in this case doesn't mean "any", like it does in every other context. Here, it should be thought as a last resort record or as if nothing exists. 

## Real subdomains
With the behavior of wildcard records only working when no record exists, we need to directly address any instance where we create a subdomain. So each time we add an A record, we are effectively creating a new subdomain which we need to address. Luckily, we can just take the same approach as before to protect the subdomain. The example below shows how that would work:

```bash
# Protection for the base domain and it non-existent subdomains
example.com.		3600	IN	TXT	"v=spf1 ip:1.2.3.4 -all"
*.example.com.		3600	IN	TXT	"v=spf1 -all"

# Create a new A record
sub.example.com.    3600    IN  A   4.3.2.1
# Protect the subdomain and non-existent subdomains of sub.example.com
sub.example.com.	3600	IN	TXT	"v=spf1 -all"
*.sub.example.com.	3600	IN	TXT	"v=spf1 -all"
```

## Conclusion
SPF is a fantastic tool. It's simple, easy to implement and doesn't really cost you anything. However, I have a feeling that due to its simplicity the SPF subdomain issue seems to be overlooked in almost every implementation I have seen. Now it's possible this is only a theoretical issue and when DKIM and DMARC are in play the chances of emails getting through this gap dramatically shrink. Furthermore, even if we create the tightest SPF implementation, it still completely relies on the receiving mail server being configured to take action on the SPF fail.

But for now, my new A record will be accompanied by 2 TXT records. 

A final note - the 4 DNS hosting services that I have worked with since learning this don't allow the creations of wildcard records. When asked why, the providers response is, "no one has asked for wildcard records before".

