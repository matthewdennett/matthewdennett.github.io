---
layout: post
title: The SPF Subdomain Issues
tags: DNS Security
---

While setting up some new DNS records for a mail gateway migration, I found myself reading the related RFCs which we all know tends to hurt the brain but this time it lead to something quite intersting. After reading the ins and outs of SPF, DKIM and DMARC I could see several common implementation scenarios that would allow carfully crafted emails to cuircumvent the implemented controlls as SPF doesn't apply to subdomains.

It might just be that 1 DNS A record needs another 2 TXT records to close the gaps. 

<!--more-->



## SPF 
SPF is a pretty straight forward consept and implementation, we cimply publish a DNS TXT record that lists the permitted mail servers for a domain. 

```
hello.com.		3600	IN	TXT	"v=spf1 ip4:1.2.3.0/24 ip4:4.3.2.1 include:spf.protection.outlook.com -all"
```

Its simple to implement and is extreamly effective at what it does, however it is worth noting a couple things...
- The action taken is competely depndant on the configuration of recieving email gateway what to do when SPF fails. 
- A missing/nonexistent SPF records are often treated as neutral result, which may be treated differently by different vendors, but is commonly treated as benine. 
- SPF lookups are for the exact domain and subdomains are **NOT** covered

The last point here, is the one to take the most note of. As sub domains are not covered by a SPF record, emails sent from a sub domain, regardless of weather the sub domain exists or not the SPF of the email will be checked against an SPF record that doesn't exists. So how do we publish a records for a nonexistan sub domain? Thats where a wildcard records can help.   

## Wildcard Records

[rfc7208](https://datatracker.ietf.org/doc/html/rfc7208#section-3.5) has a really interesting section that aludes to this issues in the example provided but, doesn't clearly articulate the problem or requirement.

With the use of wildcard records we can return results when a request is made for a nonexistant record. This allows us to start closing the gaps on the SPF subdomain issues by implementing a wildcard TXT record with either your standards SPF data or a SPF NULL v=spf1 -all" to be returned for nonexistant records. Here is an example of what that would look like.

```bash
# Protection for the base domain
example.com.		3600	IN	TXT	"v=spf1 ip:1.2.3.4 -all"

# Protect any nonexistant subdomain
*.example.com.		3600	IN	TXT	"v=spf1 -all"
```

Its important here to think about wildcard record correctly. The "*" in this case doesn't mean any liek it does in every other context. Here, it should be thought as a last resort record or as 'if nothing exists'. 

## Real subdomains
With the behaviour of wildcards records only working when no record exists we need to directly addres any instance where we create a subdomain. So time we add an A recors we are effectivly creating a new subdomain which we need to address. Luckely, we can just take the same approach as before to protect the sub domain. The example below show how that would work.

```bash
# Protection for the base domain and it nonexistant subdomains
example.com.		3600	IN	TXT	"v=spf1 ip:1.2.3.4 -all"
*.example.com.		3600	IN	TXT	"v=spf1 -all"

# Create a new A record
sub.example.com.    3600    IN  A   4.3.2.1
# Protect the subdomain and nonexistant subdomains of sub.example.com
sub.example.com.	3600	IN	TXT	"v=spf1 -all"
*.sub.example.com.	3600	IN	TXT	"v=spf1 -all"
```

## Conclusion
SPF is a fantastic tool. It's simple, easy to implement and doesn't really cost you anything. However, I have a feeling that due to its simplicity the SPF Subdomain issue seems to be overlooked in almost every implementation I have seen. Now its possible this is only a theretical issue and when DKIM and DMARC are in play the chances of emails getting through this gap shrink. Furthermore, even if we create the tightest SPF implementation it still completely relies on the recieving main server being configured to take action on the SPF fail.

But for now, my new A record will be acompanied by 2 TXT records. 

A final note - The 4 DNS hosting services that I have worked with since learning this don't alow the creations of wildcard records. When asked about why thier responces is "No one as asked for wildcard records before".

