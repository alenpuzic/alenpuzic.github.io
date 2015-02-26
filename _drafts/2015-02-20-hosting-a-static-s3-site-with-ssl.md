---
layout:     post
title:      Hosting A Static S3 Site With SSL
date:       2015-02-24 18:00:00
summary:    How to host a static site on S3 and CloudFront with SSL support. 
categories: cloud hosting
---
<!-- 3c2e30ed531f1b7bb8582131791ef67dc8f0edd6 -->

Static sites are cool. They are generally faster, less resource intensive and avoid platform lock-in. Not to mention
they're a heck of a lot more secure than their dynamic counterparts. 
 
The purpose of this article is to guide you in setting up your static site on AWS <ins>with SSL support</ins>, which is 
something I feel is imperative in today's world. Encryption should be a necessity and not just a nice-to-have. For a 
personal site you can purchase a decent SSL certificate for under $10 a year, from companies like 
[NameCheap](https://www.namecheap.com). Even better - soon , summer of 2015, you'll be able to get a free certificate
from the folks at [Let's Encrypt](https://letsencrypt.org). Let's get started.

### Overview

In this tutorial we're going to learn how to use various AWS cloud services to serve a static site that is (practically) 
infinitely scalable, very budget friendly and served over SSL/TLS. In particular, we'll use S3 cloud storage to host
our static site files, CloudFront content delivery network to distribute our content over SSL and finally Route53 
to point DNS requests to CloudFront.   
 
There's a couple of things we should understand before we start. CloudFront allows you to use an AWS provided SSL
certificate instead of using your own certificate, but it costs a whopping $600 a month to use this premium service.
That makes it a non-option for most people. Also, CloudFront is not required to serve your custom domain site from S3,
but it is required if you want to use SSL. 

_This rest of this article assumes that you already have an active AWS account. It also assumes that you have 
registered a domain name and have already generated static files necessary for your site. From this point on we're going
to use example.com as placeholder for your custom domain. Please make sure to replace example.com with your domain 
name when following the procedures outlined below._

### Create S3 buckets

To host your site on Amazon's S3 you need to create at least 2 buckets, one for your apex domain (example.com) and 
another for the www subdomain (www.example.com). We're also going to create a 3rd optional bucket (logs.example.com) 
to store CloudFront access logs, which we'll setup at a later stage of this tutorial.

_It's worth noting that bucket names in S3 are universal, so if another AWS user has a bucket 'example.com' 
then you won't be able to create a bucket with that name. If that happens don't worry. The bucket names you'll 
create do not have to match your domain name._

We need to create 3 buckets for our hosting setup:

  * example.com 
  * www.example.com
  * logs.example.com

Log into your AWS Console, proceed to the S3 Management Console and create the above listed buckets, like in the 
example shown bellow. You can set the Region option to whatever you prefer, or just leave it at the default setting. 

![Create S3 Bucket](/images/3c2e30ed531f1b7bb8582131791ef67dc8f0edd6_001.png)

### Configure S3 buckets

Next we have to configure our newly created buckets. Let's start with the apex domain bucket, example.com. This is the
bucket that will store all of our static site files. First we enable site hosting from the 'Static Website Hosting' 
section of the properties tab. 

![Enable S3 Bucket Static Site Hosting](/images/3c2e30ed531f1b7bb8582131791ef67dc8f0edd6_002.png)

Next we need to attach a bucket policy to example.com. This policy will configure bucket permissions so all of the 
files in our bucket will become publicly accessible. You can do this by clicking the 'Add Bucket Policy' button in the
'Permissions' section of your bucket's properties. 

![Attach S3 Bucket Policy](/images/3c2e30ed531f1b7bb8582131791ef67dc8f0edd6_003.png)

Here's the policy in plaintext so you can copy it. Make sure to replace example.com with your domain name.

{% highlight json %}
{
  "Version":"2012-10-17",
  "Statement":[{
	"Sid":"AddPerm",
        "Effect":"Allow",
	  "Principal": "*",
      "Action":["s3:GetObject"],
      "Resource":["arn:aws:s3:::example.com/*"
      ]
    }
  ]
}
{% endhighlight %}

Now the only thing left to do is modify our www.example.com bucket to redirect all requests to our example.com 
bucket. Once again we go to the 'Static Website Hosting' section of the bucket properties tab but this time we pick the
'Redirect all requests to another host name' option and set the value to 'example.com', as shown in the example 
screenshot below.

![Redirect S3 Requests](/images/3c2e30ed531f1b7bb8582131791ef67dc8f0edd6_004.png)

Now we're finished setting up our S3 buckets! You can now upload your static site files to the example.com bucket.
S3 will redirect all requests from www.example.com to example.com bucket. We don't have to do anything with our
'logs.example.com' bucket, yet. We'll configure access logs using CloudFront at the end of this article.

### Obtain a SSL certificate

_This section of the guide assumes you need to generate a certificate signing request (CSR) in order to obtain a SSL
certificate. If you already have your certificate files ready you can skip over to the next section. If you don't,
keep reading._

You'll need to pick a certificate authority (CA) to purchase your SSL certificate from. There's a wide range of decent,
affordable CAs out there. [GoDaddy](https://www.godaddy.com) and [NameCheap](https://www.namecheap.com) both offer
dirt cheap certificates that are good enough for personal sites. Once you purchase a certificate from a CA you'll be
asked to provide your CSR, so that the certificate can be signed properly. The command prompt example below
illustrates how to create a certificate signing request using OpenSSL: 
 
{% highlight bash %}
$ openssl req -nodes -newkey rsa:4096 -keyout example.key -out example.csr
Generating a 4096 bit RSA private key
.+++
...........................+++
writing new private key to 'example.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:Texas
Locality Name (eg, city) []:Austin
Organization Name (eg, company) [Internet Widgits Pty Ltd]:YourCompany LLC
Organizational Unit Name (eg, section) []:IT
Common Name (e.g. server FQDN or YOUR name) []:example.com
Email Address []:admin@example.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:YourCompany LLC
{% endhighlight %}

_The example above creates a 4096bit key for added security. However, some certificate authorities might ask you to
use a 2048bit key instead. You can replace 'rsa:4096' with 'rsa:2048' above to lower the decrease key size._
 
This creates two files for you, example.key or your private key, and example.csr which is your CSR. You'll upload
the contents of example.csr to your certificate authority (CA) and they'll provide you certificate files. Keep your 
private key in a safe place and don't share it with anybody!

There's one last thing we need to take care of. The private key that was generated in the step above is not in PEM 
format, which is required by CloudFront. We need to convert the key into PEM.

{% highlight bash %}
$ openssl rsa -in example.key -outform PEM -out example_pem.key
{% endhighlight %}

This creates a new key file, example_pem.key, which is formatted properly for AWS.

### Upload your SSL certificate to IAM

_This section assumes you have AWS command line interface installed on your machine. At the time of this writing this
was the only way to upload certificates to AWS/IAM._

There are 3 files required for uploading a SSL certificate to AWS:
  
  * Private key in PEM format (example_pem.key)
  * SSL certificate (provided by your CA; example_com.crt)
  * Intermediate certificates bundle file (provided by your CA; CA.ca-bundle)
  
Once we have all 3 files ready we can issue AWS IAM command to upload the certificate:

{% highlight bash %}
$ aws iam upload-server-certificate --server-certificate-name ExampleComSSLCert --certificate-body file://example_com.crt --private-key file://example_pem.key --certificate-chain file://CA.ca-bundle --path /cloudfront/
{
"ServerCertificateMetadata": {
    "ServerCertificateId": "ASCAJT3AQIK3Z--------",
    "ServerCertificateName": "ExampleGenComSSLCert",
    "Expiration": "2016-02-06T23:59:59Z",
    "Path": "/cloudfront/",
    "Arn": "arn:aws:iam::************:server-certificate/ExampleComSSLCert",
    "UploadDate": "2015-02-06T21:50:57.560Z"
    }
}
{% endhighlight %}

We've now created a copy of our SSL certificate and stored it in AWS IAM. Now we can proceed to setup CloudFront with
our S3 buckets and the SSL certificate.

### Create a CloudFront distribution

Now we need to go to the 'CloudFront Management Console' and create a new distribution. Make sure to select 'Web' as
the delivery method. 

![Redirect S3 Requests](/images/3c2e30ed531f1b7bb8582131791ef67dc8f0edd6_005.png)

Here's a list of the applicable configuration options and the values you should set. There are a lot of other options
not listed in the list below - you can leave those at their default values. 

  * Origin Domain Name: _example.com.s3.amazonaws.com_
  * Viewer Protocol Policy: _Redirect HTTP to HTTPS_
  * Allowed HTTP Methods: _GET, HEAD_
  * Object Caching: _Use Origin Cache Headers_
  * Forward Cookies: _None_
  * Forward Query Strings: _No_
  * Smooth Streaming: _No_
  * Restrict Viewer Access: _No_
  * Price Class: _Use All Edge Locations (Best Performance)_
  * Alternate Domain Names: _www.example.com example.com_
  * SSL Certificate: _ExampleComSSLCert_
  * Default Root Object: _index.html_
  * Logging: _On_
  * Bucket for logs: _logs.example.com_
  * Log Prefix: _example.com/_
  * Cookie Logging: _Off_
  * Custom SSL Client Support: _Only Clients that Support Server Name Indication (SNI)_

Allow roughly 15 minutes for the changes to propagate through CloudFront edge locations.

This step just configured logs.example.com bucket to serve as storage for site access logs. Roughly every hour
CloudFront will save access logs into the given bucket. This is a great scalable way of keeping a backup of all your
access logs and I highly recommend it.

### Configure Route53

In order to point our custom domain name to our hosted static files we'll need to setup Amazon's DNS 
service, Route53. From the AWS Console go to Route53 Management Console and click the button that says
'Create Hosted Zone'. We're going to create a new zone for our custom domain example.com.

![Redirect S3 Requests](/images/3c2e30ed531f1b7bb8582131791ef67dc8f0edd6_006.png)

Once your zone is created you'll be shown a screen with 'Hosted Zone Details' as in the example shown below.

![Redirect S3 Requests](/images/3c2e30ed531f1b7bb8582131791ef67dc8f0edd6_007.png)

Now go into your newly created hosted zone to view the record sets. You'll find that Route53 created 2 DNS records for
you, NS and SOA. The NS record specified the name servers for your domain. Make a note of the values in the NS record,
as we're going to use those later on to point our domain registrar to Route53's nameservers. The SOA record is used to specify 
authoritative information about your hosted zone. Now we need to add 2 more records to your hosted zone so point
web requests to your site's CloudFront distribution. 

Go into your hosted zone. First we're going to add a new A record that will point example.com to CloudFront. 
Create a new record of type A, set Alias to Yes and point Alias Target to the matching CloudFront distribution. 
You can leave all other options as default.

![Redirect S3 Requests](/images/3c2e30ed531f1b7bb8582131791ef67dc8f0edd6_008.png)

Next we're going to add a CNAME record type. Create a new record of type CNAME, make sure the name is prefixed with 
'www', set Alias to No and value to 'example.com'.

![Redirect S3 Requests](/images/3c2e30ed531f1b7bb8582131791ef67dc8f0edd6_009.png)

And that's it! AWS is now fully configured to host your static site with your custom SSL certificate and domain name.

### Point Your Domain to Route53

Finally, all you have left to do is point your domain name to Route53 nameservers. This can be done
through your domain name registrar. Here's links to instructions for some commonly used registrars:

  * [GoDaddy] (https://support.godaddy.com/help/article/12317/setting-custom-nameservers-for-domains-registered-with-us)
  * [NameCheap] (https://www.namecheap.com/support/knowledgebase/article.aspx/767/10/how-can-i-change-the-nameservers-for-my-domain)
  * [1and1] (https://help.1and1.com/domains-c36931/manage-domains-c79822/dns-c37586/use-your-own-name-server-for-a-1and1-domain-a594904.html)
  
### Done

You're done! You should now have a static site with your files hosted on S3, your content distributed along CloudFront
edge nodes and content served securely via SSL. 