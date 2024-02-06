---
title: Open Source Intelligence
date: 2018-12-04
layout: post
author: andrew
categories: [linux, OSCP ]
---

Being able to research and learn everthing about your target is a mission-critical aspect of any penetration test.  One of the ways to gather information about a target is through Open Source Intelligence, or OSINT.  There are two main forms of OSINT: active and passive.  Active OSINT occurs when your computer makes some sort of connection to the target computer.  Passive OSINT uses other services to gather this data.  In this post I'll walk you through the basics of Passive OSINT

## Passive Footprinting:

## Whois

Whenever you register a new domain, its required to add an entry to the Whois database. This database provides some basic information about the owners of a specific domain name.  Many services are now available, howeer, to hide sensitive private information, but this is still a good place to start your OSINT.  We can use the `whois` command found in most linux distros to look up a domain:

```
$ whois briggselsperger.com 
   Domain Name: BRIGGSELSPERGER.COM 
   Registry Domain ID: 2219218811_DOMAIN_COM-VRSN 
   Registrar WHOIS Server: whois.namecheap.com 
   Registrar URL: http://www.namecheap.com 
   Updated Date: 2018-03-30T21:57:57Z 
   Creation Date: 2018-01-27T05:00:39Z 
   Registry Expiry Date: 2019-01-27T05:00:39Z 
   Registrar: NameCheap, Inc. 
   Registrar IANA ID: 1068 
   Registrar Abuse Contact Email: abuse@namecheap.com 
   Registrar Abuse Contact Phone: +1.6613102107 
   Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited 
   Name Server: DNS1.REGISTRAR-SERVERS.COM 
   Name Server: DNS2.REGISTRAR-SERVERS.COM 
   DNSSEC: signedDelegation 
   DNSSEC DS Data: 33880 13 1 5EB96247DD93FE5C8FA104812BF2518479BB016D 
   URL of the ICANN Whois Inaccuracy Complaint Form: https://www.icann.org/wicf/ 
>>> Last update of whois database: 2018-12-04T03:19:16Z 


  [ … ] 


The Registry database contains ONLY .COM, .NET, .EDU domains and 
Registrars. 
Domain name: briggselsperger.com 
Registry Domain ID: 2219218811_DOMAIN_COM-VRSN 
Registrar WHOIS Server: whois.namecheap.com 
Registrar URL: http://www.namecheap.com 
Updated Date: 2018-03-30T21:57:57.00Z 
Creation Date: 2018-01-27T05:00:39.00Z 
Registrar Registration Expiration Date: 2019-01-27T05:00:39.00Z 
Registrar: NAMECHEAP INC 
Registrar IANA ID: 1068 
Registrar Abuse Contact Email: abuse@namecheap.com 
Registrar Abuse Contact Phone: +1.6613102107 
Reseller: NAMECHEAP INC 
Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited  
Registry Registrant ID:  
Registrant Name: WhoisGuard Protected 
Registrant Organization: WhoisGuard, Inc. 
Registrant Street: P.O. Box 0823-03411  
Registrant City: Panama 
Registrant State/Province: Panama 
Registrant Postal Code:  
Registrant Country: PA 
Registrant Phone: +507.8365503 
Registrant Phone Ext:  
Registrant Fax: +51.17057182 
Registrant Fax Ext:  
Registrant Email: 1f263bac93aa4f4e8b8908137ae06428.protect@whoisguard.com 
Registry Admin ID:  


  [ ... ]


Name Server: dns1.registrar-servers.com  
Name Server: dns2.registrar-servers.com  
DNSSEC: unsigned 
```

From the above scan we can determine:
  * DNS Servers (sometimes these may be located within the target's org)
  * Domain Name Expiry Date
  * Domain Name Registrar used

In some cases, additional information can be obtained with whois:
  * Names and/or email addresses of employees
  * Username and email formats, (ex. jdoe@company.com, john.doe@company.com)
  * Physical building locations


## Netcraft

[Netcraft](https://searchdns.netcraft.com/) is a web-based OSINT tool we can use to find the IP address of a website. It can also provide other information about the hosting provider, SPF Records, and (maybe most importantly) the tech stack of the website. We can use all of this later

![Netcraft Screenshot](/blog/assets/img/osint-netcraft.png)


## nslookup

Once we have the IP address of the target, we can dig further into the infrastructure.  A great way to get started with this is by using DNS to determine the IP address of a target.

```
$ nslookup www.briggselsperger.com 
Server:        1.1.1.1 
Address:    1.1.1.1#53 
  
Non-authoritative answer: 
Name:    www.briggselsperger.com 
Address: 18.188.50.45
```

This simple can tells us two useful tidbits of information:
  * We are using `1.1.1.1` DNS nameserver (Thanks, Cloudflare!)
  * The target's website IP is `18.188.50.45`

## Whois (Reloaded)

Now that we have an IP address to get started, we can run whois again on that IP to gather even more information about the target:

```
$ whois 18.188.50.45 

  [ ... ]  

NetRange:       18.128.0.0 - 18.255.255.255 
CIDR:           18.128.0.0/9 
NetName:        AT-88-Z 
NetHandle:      NET-18-128-0-0-1 
Parent:         NET18 (NET-18-0-0-0-0) 
NetType:        Direct Allocation 
OriginAS:        
Organization:   Amazon Technologies Inc. (AT-88-Z) 
RegDate:        2018-06-29 
Updated:        2018-09-19 
Ref:            https://rdap.arin.net/registry/ip/18.128.0.0 
  
  
OrgName:        Amazon Technologies Inc. 
OrgId:          AT-88-Z 
Address:        410 Terry Ave N. 
City:           Seattle 
StateProv:      WA 
PostalCode:     98109 
Country:        US 
RegDate:        2011-12-08 
Updated:        2017-01-28 
Comment:        All abuse reports MUST include: 
Comment:        * src IP 
Comment:        * dest IP (your IP) 
Comment:        * dest port 
Comment:        * Accurate date/timestamp and timezone of activity 
Comment:        * Intensity/frequency (short log extracts) 
Comment:        * Your contact details (phone and email) Without these we will be unable to identify the correct owner of the IP address at that point in time. 
Ref:            https://rdap.arin.net/registry/entity/AT-88-Z 
  
  
OrgNOCHandle: AANO1-ARIN 
OrgNOCName:   Amazon AWS Network Operations 
OrgNOCPhone:  +1-206-266-4064  
OrgNOCEmail:  amzn-noc-contact@amazon.com 
OrgNOCRef:    https://rdap.arin.net/registry/entity/AANO1-ARIN 
  
OrgTechHandle: ANO24-ARIN 
OrgTechName:   Amazon EC2 Network Operations 
OrgTechPhone:  +1-206-266-4064  
OrgTechEmail:  amzn-noc-contact@amazon.com 
OrgTechRef:    https://rdap.arin.net/registry/entity/ANO24-ARIN 
  
OrgAbuseHandle: AEA8-ARIN 
OrgAbuseName:   Amazon EC2 Abuse 
OrgAbusePhone:  +1-206-266-4064  
OrgAbuseEmail:  abuse@amazonaws.com 
OrgAbuseRef:    https://rdap.arin.net/registry/entity/AEA8-ARIN 
```

This new scan gives us two more pieces of information:
  * Who owns the server
  * What IP Ranges the target owns



