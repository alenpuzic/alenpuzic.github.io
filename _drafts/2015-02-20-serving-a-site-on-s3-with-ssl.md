---
layout:     post
title:      Serving A Site On S3 With SSL
date:       2015-02-20 18:00:00
summary:    How to host a static site on S3 and CloudFront with SSL support. 
categories: jekyll pixyll
---
<!-- 3c2e30ed531f1b7bb8582131791ef67dc8f0edd6 -->

Static sites are cool. They are generally faster, less resource intensive and avoid platform lock-in. Not to mention
they're a heck of a lot more secure than their dynamic counterparts. For a more in depth look at the pros and cons of
static sites you can read my post on [Benefits of Static Sites]({% post_url 2014-06-08-pixyll-has-pagination %}).
 
The purpose of this article is to guide you in hosting your site on AWS __with__ SSL support, which is something I feel is 
imperative. In today's world encryption is something that should be a necessity and not just a nice-to-have. For a 
personal site you can purchase a decent SSL certificate for as little as $9 month, from the likes of 
[Comodo](https://www.comodo.com). Even better - soon , summer of 2015, you'll be able to get a free certificate
from the folks at [Let's Encrypt](https://letsencrypt.org). Let's get started.

### Overview

Blah blah blah.

_This rest of this article assumes that you already have an active AWS account. It also assumes that you have 
registered a domain name and have already generated the static files for your site._

### Create S3 buckets


To setup your static site with a custom domain on Amazon's S3 you need to create at least 2 buckets, one
for your apex domain (example.com) and another for the www subdomain (www.example.com). We're also going to create a 3rd
optional bucket (logs.example.com) to store CloudFront access logs, which we'll setup at a later stage of this tutorial.

_It's worth noting that bucket names
in S3 are universal, so if another AWS user has a bucket 'example.com' then you won't be able to create a bucket with
that name. If that happens don't worry. The bucket names we're going to use for your static site do not have to
match your domain name._

We need to create 3 buckets for our hosting setup:

  * example.com 
  * www.example.com
  * logs.example.com

Log into your AWS Console, proceed to the S3 Management Console and create the above listed buckets.

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

Now we're finished setting up our S3 buckets! Bucket 'example.com' will contain all our static site files and all 
requests to 'www.example.com' will get redirected to 'example.com' bucket by S3. We don't have to do anything with our
'logs.example.com' bucket. We'll configure access logs using CloudFront at the end of this article. 

### Configure Route53

In order to host our static site on S3/CloudFront with a custom domain name we'll need to setup use Amazon's DNS 
service, or Route53. From within the AWS Console go to Route53 Management Console and click the button that says
'Create Hosted Zone'. We're going to create a new zone for our custom domain example.com.

![Redirect S3 Requests](/images/3c2e30ed531f1b7bb8582131791ef67dc8f0edd6_005.png)

Once your zone is created you'll be shown a screen with 'Hosted Zone Details' as in the example shown below.

![Redirect S3 Requests](/images/3c2e30ed531f1b7bb8582131791ef67dc8f0edd6_006.png)

Now go into your newly created hosted zone to view the record sets. You'll find that Route53 created 2 DNS records for
you, NS and SOA. The NS record specified the name servers for your domain. Make a note of the values in the NS record,
as we're going to use those later on to point our domain to Route53's nameservers. The SOA record is used to specify 
authoritative information about your hosted zone. We're going to need to create two new records (A and CNAME records)
but before we do that we must set up CloudFront to serve our website through it's content delivery network.

### Obtain a SSL certificate

_This section of the guide assumes you need to generate a certificate signing request (CSR) in order to obtain a SSL
certificate. If you already have your certificate files ready you can skip over to the next section. If you don't,
keep reading._

You'll need to pick a certificate authority (CA) to purchase your SSL certificate from. There's a wide range of decent,
affordable CAs out there. [GoDaddy](https://www.godaddy.com) and [NameCheap](https://www.namecheap.com) both offer
dirt cheap certificates that are good enough for personal sites. Once you purchase a certificate from a CA you'll be
asked to provide your CSR, so that the certificate can be signed properly. The command prompt example below
illustrates how to create a certificate signing request on a Linux box running Ubuntu OS. 
 
{% highlight bash %}
$ openssl req -nodes -newkey rsa:2048 -keyout example.key -out example.csr
Generating a 2048 bit RSA private key
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
 
This creates two files for you, example.key or your private key, and example.csr which is your CSR. You'll provide
the contents of example.csr to your certificate authority (CA) and they'll provide you with one or more certificate 
files. 

There's one last thing we need to take care of. The private key that was generated in the step above is not in PEM 
format, which is required by CloudFront. We need to convert the key into PEM.

{% highlight bash %}
$ openssl rsa -in example.key -outform PEM -out example_pem.key
{% endhighlight %}

This creates a new key file, example_pem.key, which is formatted properly for AWS.

### Upload your SSL certificate to CloudFront

_This section assumes you have AWS command line interface installed on your machine. At the time of this writing this
was the only way to upload certificates to AWS._

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

We've now created a copy of our SSL certificate and stored it in AWS' IAM. Now we can proceed to setup CloudFront with
our S3 buckets and the SSL certificate.

### Create a CloudFront distribution

Now we need to go to the 'CloudFront Management Console' and create a new distribution. Make sure to select 'Web' as
the delivery method. 

![Redirect S3 Requests](/images/3c2e30ed531f1b7bb8582131791ef67dc8f0edd6_007.png)

Here's a list of the applicable configuration options and the values you should set.  

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

Allow roughly 15 minutes for the changes to propagate through the CloudFront.

### Point Route53 to CloudFront

The last piece of the AWS puzzle is to configure your Route53 hosted zone to point to the CloudFront distribution you
just created. Go back into Route53 and go into your hosted zone. As mentioned earlier we're going to need to add
two more DNS records in order to host your static site. 

First we're going to add a new A record that will point example.com to CloudFront:

![Redirect S3 Requests](/images/3c2e30ed531f1b7bb8582131791ef67dc8f0edd6_009.png)

Next we're going to add a CNAME record type:

![Redirect S3 Requests](/images/3c2e30ed531f1b7bb8582131791ef67dc8f0edd6_009.png)

And that's it! AWS is now fully configured to host your static site with your custom SSL certificate and domain name.

### Point Your Domain to Route53

Now you have to make your changes public by pointing your domain name to use Route53's nameservers. This can be done
through your domain name registrar. 

Here's links to instructions on how to do this on common registars:

  * [GoDaddy] (https://support.godaddy.com/help/article/12317/setting-custom-nameservers-for-domains-registered-with-us)
  * [NameCheap] (https://www.namecheap.com/support/knowledgebase/article.aspx/767/10/how-can-i-change-the-nameservers-for-my-domain)
  * [1and1] (https://help.1and1.com/domains-c36931/manage-domains-c79822/dns-c37586/use-your-own-name-server-for-a-1and1-domain-a594904.html)