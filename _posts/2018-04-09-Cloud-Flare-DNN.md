---
layout: post
title:  CloudFlare for DNN
modified: 2018-04-09 08:00:00
categories: DNN
tags: [Cloudflare, Security, https, SSL, TLS, DNN, CDN, Caching, Performance]
share: true
comments: true
---
I was helping my friend [Clint Patterson](https://twitter.com/CBPSC) deploy a new DNN website and recommended that we enforce HTTPS even though for the small website he really didn't need it. With the ease of tools such as [CloudFlare](https://www.cloudflare.com) it is really easy to set up and enforce https which not only makes your site secure but makes your users feel comfortable with the lock icon displaying in the browser. 

Why CloudFlare? 

| Pros                                          | Cons                                                  |
|-----------------------------------------------|-------------------------------------------------------|
| Free SSL Certificate                          | You need to use CloudFlare as your nameserver         |
| Free CDN just by signing up                   | You have to use CloudFlare to configure your DNS Records     |
| Free Static Content minification              | You have to get over [CloudBleed](https://blog.cloudflare.com/incident-report-on-memory-leak-caused-by-cloudflare-parser-bug/)                       |
| Built in advanced Cryptography features       ||
| DNN Performance Increase                      ||
| Search Engine Optimization                    ||

<br />

This isn't a complete list but some things to start thinking about

I try to configure all of my websites with cloudflare because the SSL support and CDN are both huge and the performance increase is apparent after the switch for DNN websites.

## CloudFlare Pre-Reqs ##
While there is no real Pre-Reqs to get started with cloudflare I have a small checklist I go through that simplifies issues that may arise.

Does my website do any of the following

* Serve any HTTP routes?
* Reference external routes or libraries such as a javascript library that are linked over HTTP
* Serve any images that are external links over HTTP
* Do anything of HTTP

There is a trend here on my checklist: 

* "Does my website do anything over HTTP or unsecured traffic?"

If you answered yes to any of these questions, go ahead and fix all of them before moving on. If you do not you will run into issues with your CloudFlare configuration on most popular browsers which will return a Mixed Content Error

### Mixed Content Errors ###
What is a Mixted Content Error?

This is a fancy word that your website was connected via https and is secure but links on your website such as images, anchors or static content were served unencrypted over http. You should fix this now and not have to deal with it later.

## Configure CloudFlare ##
Start off by going to [CloudFlare](https://www.cloudflare.com/) and creating a free account. Once you are done creating your account we are ready to add your site:

1. Click + Add Site in the navigation bar
2. Enter the name of your site

CloudFlare will now query your DNS records to make your transition as simple as possible. This means it will try and determine all of your DNS records so you don't need to enter them.

Before you continue I suggest you have all of your records backed up just in case something goes wrong. It'll also be handy if not all of your records are copied over.

3. Make sure you select the FREE Plan unless you need something a little bit more powerful
4. The next screen shows all of your DNS records, you can add new ones if you want. We can always change this later
5. Update your nameservers: This will vary depending on your domain registrar, but copy the nameservers provided by the cloudflare setup

<strong>That's IT!</strong>

Really, that is all you need to do to setup CloudFlare and all of your traffic has now been optimized. 

## Setup SSL Certificate ##
We still have one last thing to do before we can say we are really done, we need to set up our SSL Certificate and enforce HTTPS. 

Navigate to your newly configured website's dashboard on the CloudFlare interface. You should see a list of controls at the top. We want to select Crypto

![Cloud Flare Nav Bar]({{ site.url }}/assets/posts/2018-04/cloud-flare-navbar.png)

Complete the following steps:

* SSL Mode
* Create Certificate
* Enforce HTTPS

### SSL Mode ###
At the top of the crypto page you will see a an option to change the type of SSL mode we are using on our website.

You want to use <strong>Full</strong>

This means that you want all of your traffic to be served over HTTPS

### Create Certificate ###
Scrolling down the page you will see a section called Origin Certificates with a button labeled "Create Certificate". This is not to be confused with "Edge Certificates". See the screenshot below:

![Cloud Flare Create Certificate]({{ site.url }}/assets/posts/2018-04/cloud-flare-create-cert.png)

After selecting Create Certificate you will be asked some general questions about your certificate. For the most part you can leave everything filled in

Click the next button and your certificate will be generated. You should see the following information:

* Certificate
* Private Key

This is where things get tricky for DNN, you need to copy the contents of your Private Key and install it onto your IIS Instance. This configuration varies with just about every hosting provider so it may be best to contact support and let them know what you are trying to do. They should be able to help you.

Remember, the private key is generated only once so if you lose it, you will have to regenerate your certificate.

#### Certificate Status ####
In my experience it takes about 1-60 minutes for the certificate to start working correctly. CloudFlare documentation states it may take up to 24 hours. It typically happens in about 15 minutes for me.

### Enforce HTTPS ###
As you scroll through the different Crypto Settings in CloudFlare you will notice the following settings:

* Always use HTTPS
* Automatic HTTPS Rewrites

I recommend turning both of these on. They will force every connection to your site to be through a HTTPS connection and if something is served from your site for some reason as HTTP it will re-write it to use HTTPS.

The HTTPS Rewrite feature doesn't always work, but it is a good idea as a safe guard

## That's Really It! ##
After completing everything documented here you should have a DNN instance running with SSL and using CloudFlare. You should start to notice secure traffic and much faster load times.