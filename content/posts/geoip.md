---
title: "Find geolocation of an IP address"
date: 2020-06-05
categories: [prog,sec,sysadmin]
tags: [go,shell]
---

I wrote a little tool called [geoip](https://github.com/jreisinger/geoip) that finds out geographic location of an IP address. It's using the MaxMind's free [GeoLite2 City database](https://dev.maxmind.com/geoip/geoip2/geolite2/).

```
> geoip 1.1.1.1
city unknown, Australia, AU

> geoip $(dig +short reisinge.net)
Frankfurt am Main, Germany, DE

> geoip $(curl -s util.reisinge.net/addr)
Partizanska lupca, Slovakia, SK
```
