---
title: "Find geolocation of an IP address"
date: 2020-06-05
categories: [prog,sec,sysadmin]
tags: [go,shell]
---

I wrote a little tool called [checkip](https://github.com/jreisinger/checkip) that finds out geographic location of an IP address. It's using the MaxMind's free [GeoLite2 City database](https://dev.maxmind.com/geoip/geoip2/geolite2/).

```
> checkip 1.1.1.1
city unknown, Australia, AU

> checkip $(dig +short reisinge.net)
Frankfurt am Main, Germany, DE

> checkip $(curl -s util.reisinge.net/addr)
Partizanska lupca, Slovakia, SK
```