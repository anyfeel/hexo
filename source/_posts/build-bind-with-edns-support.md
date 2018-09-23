---
title: build bind with edns support
date: 2018-09-23 18:21:51
tags: DNS
---

## download and extract BIND.
```
$ wget ftp://ftp.isc.org/isc/bind9/9.9.3/bind-9.9.3.tar.gz
$ tar xf bind-9.9.3.tar.gz
$ cd bind-9.9.3
```

## Download the patch from [Wilmer van der Gaast](http://wilmer.gaa.st/edns-client-subnet/).
```
$ wget http://wilmer.gaa.st/edns-client-subnet/bind-9.9.3-dig-edns-client-subnet-iana.diff
```

## Patch the code, configure (without OpenSSL because we only want *dig*) and compile.
```
$ patch -p0 < bind-9.9.3-dig-edns-client-subnet-iana.diff
$ ./configure --without-openssl
$ make
```

## Now you will have *dig* placed in bin/dig. You can try it this way:
```
$ ./bin/dig/dig @ns1.google.com www.google.es +client=157.88.0.0/16
```

Note the CLIENT-SUBNET line in the answer OPT PSEUDOSECTION.>
