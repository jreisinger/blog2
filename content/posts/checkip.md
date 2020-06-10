---
title: "Find out information about an IP address"
date: 2020-06-05
categories: [prog,sec,sysadmin]
tags: [go,shell]
---

I wrote a little tool called [checkip](https://github.com/jreisinger/checkip) that finds information about an IP address.

```
> checkip 1.1.1.1
Geo (maxmind.com): city unknown, Australia, AU
ASN (iptoasn.com): 7497, 1.1.1.1 - 1.1.1.1, CSTNET-AS-AP Computer Network Information Center, CN

> checkip $(dig +short google.com)
Geo (maxmind.com): city unknown, United States, US
ASN (iptoasn.com): 15169, 172.217.0.0 - 172.217.31.255, GOOGLE - Google LLC, US

> checkip $(curl -s util.reisinge.net/addr) # your own IP address
Geo (maxmind.com): Bratislava, Slovakia, SK
ASN (iptoasn.com): 15962, 109.230.0.0 - 109.230.63.255, OSK-DNI Slovakia, SK
```
