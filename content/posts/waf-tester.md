---
title: "waf-tester: testing Web Application Firewalls"
date: 2020-03-12
categories: [prog,sec]
tags: [sysadmin,go,shell,docker,waf]
---

I've been working with WAFs for some time and to make my work easier I created couple of tools. They are inside the [waf-tester](https://github.com/jreisinger/waf-tester) project. Let's see how I use these tools.

## WAF tests

`waf-tester` binary executes and then evaluates tests against a specified host. So what are these tests. They are basically HTTP requests and expected responses represented in YAML format. For example:

```
  - test_title: RCE
    stages:
      - stage:
          input:
            method: "GET"
            uri: "?exec=/bin/bash"
          output:
            status: 403
```

This means that `waf-tester` will make a GET request with this URL `<scheme>://<host>/?exec=/bin/bash` (`scheme` and `host` are command line flags) and it will expect status 403 in the response. If this expectation proves true the test passed (`OK`). Otherwise it failed (`FAIL`).

There exist additional `input` and `output` fields. You can use different methods, insert custom headers or instead of checking the request code you can check the WAF logs:

```
  - test_title: 911100-5
    stages:
      - stage:
          input:
            method: "TEST"
            port: 80
            headers:
              User-Agent: "ModSecurity CRS 3 Tests"
              Host: "localhost"
          output:
            log_contains: "id \"911100\""
```

The YAML format is based on [FTW](https://github.com/CRS-support/ftw/blob/master/docs/YAMLFormat.md) but some fields are missing. Others, like `dest_addr`, are ignored. See the [code](https://github.com/jreisinger/waf-tester/blob/master/yaml/types.go) for details.

## WAF rules

WAF rules (or signatures) is what helps WAFs to distinguish between legitimate and malicious requests. Let's say that I want to run some tests against a [NAXSI](https://github.com/nbs-system/naxsi) WAF with default rules. To run this WAF locally in a container I execute this:

```
$ ./run-waf -s nginx/naxsi
--> Create a temporary directory
--> Copy all needed files to /tmp/tmp.yC7gNgJStX
--> Create directories for logs in /tmp
--> Run docker images for WAF and protected web server
Creating network "tmpyc7gngjstx_default" with the default driver
Building testing-waf
Creating tmpyc7gngjstx_testing-webserver_1 ... done
Creating tmpyc7gngjstx_testing-waf_1       ... done
--> Sleeping until Ctrl-C
```

`nxingx/naxsi` folder contains the `Dockerfile` and all needed files like `nginx.conf`, `naxsi.conf` and `naxsi_core.rules` that get copied into the container.

Then I open another terminal and run basic tests against the WAF (to compile and install the `waf-tester` binary run `make install` from within the [waf-tester repo](https://github.com/jreisinger/waf-tester)):

```
$ waf-tester -host localhost -scheme http
FAIL	RCE                  GET       http://localhost/?exec=/bin/bash
OK	    SQLi                 GET       http://localhost/?id=1'%20or%20'1'%20=%20'
OK	    LFI                  GET       http://localhost/?page=/etc/passwd
OK	    XSS                  GET       http://localhost/?<script>
FAIL	Scanner detection    GET       http://localhost/AppScan_fingerprint/MAC_ADDRESS_01234567890.html?9ABCDG1
OK	    Session fixation     GET       http://localhost/foo.php?bar=blah%3Cscript%3Edocument.cookie=%22sessionid=1234;%20domain=.example.dom%22;%3C/script%3E
```

You can see `FAIL` status for two tests. This means that the WAF didn't block these two requests. Let's have a more detailed view on the first failing test:

```
$ waf-tester -host localhost -scheme http -title RCE -verbose
FAIL	RCE                  GET       http://localhost/?exec=/bin/bash
  DESC       
  FILE       tests/basic-tests.yaml
  STATUS     200 OK
  CODE       200
  EXP_CODES  [403]
  EXP_LOGS   
  EXP_NOLOGS 
  EXP_ERR    false
  ERROR      <nil>
  DATA       []
  HEADERS    
    waf-tester-id: cb0338a4-d88e-bba1-7177-cb9f026ae7f8
  LOGS   
```

As you can see the test expected 403 code (`EXP_CODES`) but got 200 (`CODE`). That's why it failed.

Let's try to add (a rather naïve) custom WAF rule to fix the failing test:

```
# nginx/naxsi/naxsi_custom.rules
BasicRule "str:/bin/bash" "msg:RCE" "mz:ARGS" "s:$UWA:4" id:10001;
```

Now I rebuild the WAF container: I hit `Ctrl-C` in the first terminal and run `./run-waf -s nginx/naxsi` again. When I re-run the test you can see it's `OK` now:

```
$ waf-tester -host localhost -scheme http -title RCE
OK	RCE                  GET       http://localhost/?exec=/bin/bash
```
