---
layout: post
title: 'Making your own CA work that works with Google Chrome '
date: '2018-02-03T15:04:00.000+01:00'
author: Niels Basjes
tags: 
modified_time: '2018-02-14T20:44:18.433+01:00'
blogger_id: tag:blogger.com,1999:blog-2071414079220000885.post-8015861638960351103
blogger_orig_url: http://blog.basjes.nl/2018/02/making-your-own-ca-work-that-works-with.html
---
This post is about one single aspect of running your own CA (usually for internal systems): Making it actually work with Google Chrome.
The reason I writing this down is because I found all the other sources of information around the world very confusing.

The main problem is twofold:
* Newer browsers (like Google Chrome) require that the "Subject Alternative Name" has been defined as part of the certificate.
* When simply signing a certificate request (which includes a "Subject Alternative Name") with openssl this property is lost and the resulting certificate doesn't work. 

### Create your own CA.
On CentOS 7 (which I use right now) the openssl package contains the /etc/pki/tls/misc/CA script that makes doing this very easy.

### Generate key and create a certificate request.
Now the confusing part here is that it doesn't matter if the "Subject Alternative Name" is or isn't present in the certificate request. 

If the "Subject Alternative Name" is present in the certificate request it will be ignored anyway.

So you can create the key and certificate request with any existing system and manual.

Three basic ways for doing this:
Some systems already have one and/or is capable of generating one as part of the solution (like I have seen in VMware ESXi). So this part is done for you.
Simply using the script provided with openssl for this purpose

{% highlight bash %}
/etc/pki/tls/misc/CA -newreq
{% endhighlight %}

This will create  newkey.pem and newreq.pem
Manually via a config file.
So if you create a config file with something like this:

{% highlight ini %}
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
[ dn ]
C=NL
ST=Noord Holland
L=Amstelveen
O=Basjes
OU=At home
emailAddress=certificates@example.com
CN = server.example.com
{% endhighlight %}

then with this config file (let's call it server.example.com.config) you can do 

{% highlight bash %}
openssl req -new -sha256 -nodes \
        -out server.example.com.csr \
        -newkey rsa:2048 \
        -keyout server.example.com.key \
        -config server.example.com.config
{% endhighlight %}

Now sign the request and add the desired "Subject Alternative Name" values
To do this we create this config file where we include all the hostnames and IP addresses we want to allow (let's call it server.example.com.sanconfig).

{% highlight ini %}
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = server.example.com
DNS.2 = www.server.example.com
IP = 10.11.12.13
{% endhighlight %}

Now we sign this where we add the config file (-extfile) and instruct to use the extra settings present under v3_req (-extensions). The actual name of the "v3_req" section doesn't really matter.

{% highlight bash %}
openssl ca -policy policy_anything \
       -out server.example.com.crt \
       -in server.example.com.csr \
       -extfile server.example.com.sanconfig \
                 -extensions v3_req
{% endhighlight %}

If during the signing you see something like this then everything went right.

{% highlight bash %}
Certificate Details:
    ...
    X509v3 extensions:
        ...
        X509v3 Subject Alternative Name: 
            DNS:server.example.com, DNS:www.server.example.com, IP Address:10.11.12.13
{% endhighlight %}

As you see the signing command is pretty standard and really only adds the config file with the SAN extension.

### Last remark
And to actually make your own CA work with chrome you have to import the certificate of your own authority /etc/pki/CA/cacert.pem file into chrome as a new authority.

